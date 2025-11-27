# Recife Complete Examples

This document provides complete, runnable examples of Recife specifications.

## Example 1: Simple Clock

A basic clock that increments hours from 0 to 23.

```clojure
(ns examples.simple-clock
  (:require [recife.core :as r]
            [recife.helpers :as rh]))

;; ====================
;; State
;; ====================

(def initial-state
  {::hour 0})

;; ====================
;; Process
;; ====================

(r/defproc ^:fair tick
  "Increment hour, stop at 23"
  (fn [db]
    (if (< (::hour db) 23)
      (update db ::hour inc)
      (r/done))))  ;; Mark complete to avoid deadlock

;; ====================
;; Invariants
;; ====================

(rh/definvariant hour-in-valid-range
  "Hour must be between 0 and 23"
  [{::keys [hour]}]
  (<= 0 hour 23))

;; ====================
;; Properties
;; ====================

(rh/defproperty eventually-reaches-10
  "Clock eventually reaches hour 10"
  [{::keys [hour]}]
  (rh/eventually (= hour 10)))

(rh/defproperty eventually-reaches-23
  "Clock eventually completes at hour 23"
  [{::keys [hour]}]
  (rh/eventually (= hour 23)))

;; ====================
;; Model Execution
;; ====================

(defn run-clock-spec []
  @(r/run-model
     initial-state
     #{tick
       hour-in-valid-range
       eventually-reaches-10
       eventually-reaches-23}
     :trace-example true))

;; Test it
(comment
  (def result (run-clock-spec))
  (:success? result)
  (count (:trace result))
  )
```

## Example 2: Payment Activation System

Models a payment activation workflow with failure scenarios.

```clojure
(ns examples.payment-activation
  (:require [recife.core :as r]
            [recife.helpers :as rh]))

;; ====================
;; State
;; ====================

(def initial-state
  {::paid? (r/one-of #{true false})  ;; Non-deterministic
   ::status :inactive
   ::retries 0})

;; ====================
;; Processes
;; ====================

(r/defproc ^:fair activate
  "Attempt to activate service based on payment status"
  {:local {:pc ::try-activation}}

  (fn [db local]
    (case (:pc local)

      ::try-activation
      (if (::paid? db)
        ;; Paid: activate immediately
        [(assoc db ::status :activated)
         (r/goto ::done)]
        ;; Not paid: go to payment flow
        [db (r/goto ::process-payment)])

      ::process-payment
      (if (< (::retries db) 3)
        ;; Try payment
        [(update db ::retries inc)
         (r/goto ::check-payment)]
        ;; Max retries: fail
        [(assoc db ::status :failed)
         (r/goto ::done)])

      ::check-payment
      ;; Non-deterministically succeed or fail
      (let [payment-succeeded? (r/one-of #{true false})]
        (if payment-succeeded?
          [(-> db
               (assoc ::paid? true)
               (assoc ::status :activated))
           (r/goto ::done)]
          ;; Failed: retry
          [db (r/goto ::process-payment)]))

      ::done
      [db (r/done)])))

;; ====================
;; Invariants
;; ====================

(rh/definvariant status-is-valid
  "Status must be one of the expected values"
  [{::keys [status]}]
  (contains? #{:inactive :activated :failed} status))

(rh/definvariant activated-only-if-paid
  "Cannot be activated without payment"
  [{::keys [status paid?]}]
  (if (= status :activated)
    paid?
    true))

(rh/definvariant reasonable-retry-count
  "Retries should not exceed maximum"
  [{::keys [retries]}]
  (<= 0 retries 3))

;; ====================
;; Properties
;; ====================

(rh/defproperty eventually-reaches-terminal-state
  "System eventually reaches activated or failed state"
  [{::keys [status]}]
  (rh/eventually (or (= status :activated)
                     (= status :failed))))

(rh/defproperty paid-users-eventually-activated
  "If paid, eventually activated"
  [{::keys [paid? status]}]
  (if paid?
    (rh/eventually (= status :activated))
    true))

;; ====================
;; Model Execution
;; ====================

(defn run-payment-spec []
  @(r/run-model
     initial-state
     #{activate
       status-is-valid
       activated-only-if-paid
       reasonable-retry-count
       eventually-reaches-terminal-state
       paid-users-eventually-activated}
     :trace-example true))

;; Test it
(comment
  (def result (run-payment-spec))
  (:success? result)

  ;; View trace
  (doseq [[idx state] (map-indexed vector (:trace result))]
    (println "State" idx ":" state))
  )
```

## Example 3: Account Transfer (Money Conservation)

Models bank account transfers with conservation invariants.

```clojure
(ns examples.account-transfer
  (:require [recife.core :as r]
            [recife.helpers :as rh]))

;; ====================
;; State
;; ====================

(def initial-state
  {::account-a (r/one-of #{50 100})
   ::account-b (r/one-of #{50 100})
   ::total nil
   ::last-transfer nil})

(defn init-with-total [state]
  "Initialize total from account balances"
  (assoc state ::total (+ (::account-a state)
                          (::account-b state))))

;; ====================
;; Processes
;; ====================

(r/defproc ^:fair transfer-a-to-b
  "Transfer 30 from A to B"
  (fn [db]
    (when (>= (::account-a db) 30)  ;; Guard: sufficient balance
      (-> db
          (update ::account-a - 30)
          (update ::account-b + 30)
          (assoc ::last-transfer {:from :a :to :b :amount 30})))))

(r/defproc ^:fair transfer-b-to-a
  "Transfer 30 from B to A"
  (fn [db]
    (when (>= (::account-b db) 30)  ;; Guard: sufficient balance
      (-> db
          (update ::account-b - 30)
          (update ::account-a + 30)
          (assoc ::last-transfer {:from :b :to :a :amount 30})))))

(r/defproc ^:fair complete
  "Mark system as complete after some transfers"
  (fn [db]
    (when (::last-transfer db)  ;; At least one transfer happened
      (r/done))))

;; ====================
;; Constraints
;; ====================

(rh/defconstraint limit-state-space
  "Limit exploration to reasonable account values"
  [db]
  (and (<= (::account-a db) 200)
       (<= (::account-b db) 200)))

;; ====================
;; Invariants
;; ====================

(rh/definvariant type-correctness
  "Accounts and total are natural numbers"
  [{::keys [account-a account-b total]}]
  (and (nat-int? account-a)
       (nat-int? account-b)
       (nat-int? total)))

(rh/definvariant money-conserved
  "Total money is always conserved"
  [{::keys [account-a account-b total]}]
  (= (+ account-a account-b) total))

(rh/definvariant no-negative-balance
  "Balances never go negative"
  [{::keys [account-a account-b]}]
  (and (>= account-a 0)
       (>= account-b 0)))

;; ====================
;; Properties
;; ====================

(rh/defproperty transfer-eventually-happens
  "At least one transfer eventually happens"
  [db]
  (rh/eventually (some? (::last-transfer db))))

;; ====================
;; Model Execution
;; ====================

(defn run-transfer-spec []
  (let [initial (init-with-total initial-state)]
    @(r/run-model
       initial
       #{transfer-a-to-b
         transfer-b-to-a
         complete
         limit-state-space
         type-correctness
         money-conserved
         no-negative-balance
         transfer-eventually-happens}
       :trace-example true)))

;; Test it
(comment
  (def result (run-transfer-spec))
  (:success? result)

  ;; View money conservation across trace
  (doseq [state (:trace result)]
    (println "A:" (::account-a state)
             "B:" (::account-b state)
             "Total:" (::total state)
             "Sum:" (+ (::account-a state) (::account-b state))))
  )
```

## Example 4: Mutex Lock (Mutual Exclusion)

Models a simple mutex lock with multiple processes.

```clojure
(ns examples.mutex-lock
  (:require [recife.core :as r]
            [recife.helpers :as rh]))

;; ====================
;; State
;; ====================

(def initial-state
  {::lock-holder nil
   ::p1-state :idle
   ::p2-state :idle})

;; ====================
;; Processes
;; ====================

(r/defproc ^:fair process-1
  "Process 1 attempts to acquire lock"
  (fn [db]
    (case (::p1-state db)

      :idle
      (assoc db ::p1-state :trying)

      :trying
      (when (nil? (::lock-holder db))  ;; Guard: lock available
        (-> db
            (assoc ::lock-holder :p1)
            (assoc ::p1-state :critical)))

      :critical
      (-> db
          (assoc ::lock-holder nil)
          (assoc ::p1-state :idle)))))

(r/defproc ^:fair process-2
  "Process 2 attempts to acquire lock"
  (fn [db]
    (case (::p2-state db)

      :idle
      (assoc db ::p2-state :trying)

      :trying
      (when (nil? (::lock-holder db))  ;; Guard: lock available
        (-> db
            (assoc ::lock-holder :p2)
            (assoc ::p2-state :critical)))

      :critical
      (-> db
          (assoc ::lock-holder nil)
          (assoc ::p2-state :idle)))))

;; ====================
;; Invariants
;; ====================

(rh/definvariant mutual-exclusion
  "Only one process can be in critical section"
  [{::keys [p1-state p2-state]}]
  (not (and (= p1-state :critical)
            (= p2-state :critical))))

(rh/definvariant lock-holder-matches-state
  "Lock holder must be in critical section"
  [{::keys [lock-holder p1-state p2-state]}]
  (case lock-holder
    :p1 (= p1-state :critical)
    :p2 (= p2-state :critical)
    nil true))

;; ====================
;; Properties
;; ====================

(rh/defproperty p1-eventually-enters-critical
  "Process 1 eventually enters critical section"
  [{::keys [p1-state]}]
  (rh/eventually (= p1-state :critical)))

(rh/defproperty p2-eventually-enters-critical
  "Process 2 eventually enters critical section"
  [{::keys [p2-state]}]
  (rh/eventually (= p2-state :critical)))

(rh/defproperty lock-cycles
  "Lock is acquired and released infinitely often"
  [{::keys [lock-holder]}]
  (rh/always (rh/eventually (some? lock-holder))))

;; ====================
;; Model Execution
;; ====================

(defn run-mutex-spec []
  @(r/run-model
     initial-state
     #{process-1
       process-2
       mutual-exclusion
       lock-holder-matches-state
       p1-eventually-enters-critical
       p2-eventually-enters-critical
       lock-cycles}
     :trace-example true))

;; Test it
(comment
  (def result (run-mutex-spec))
  (:success? result)

  ;; View lock transitions
  (doseq [[idx state] (map-indexed vector (:trace result))]
    (println "State" idx ":"
             "Lock:" (::lock-holder state)
             "P1:" (::p1-state state)
             "P2:" (::p2-state state)))
  )
```

## Example 5: Request-Response System with Timeouts

Models a request-response system with timeout handling.

```clojure
(ns examples.request-response
  (:require [recife.core :as r]
            [recife.helpers :as rh]))

;; ====================
;; State
;; ====================

(def initial-state
  {::pending-requests #{}
   ::completed-requests #{}
   ::timed-out-requests #{}
   ::request-id 0
   ::time 0})

;; ====================
;; Processes
;; ====================

(r/defproc ^:fair submit-request
  "Submit a new request"
  (fn [db]
    (when (< (::request-id db) 3)  ;; Limit requests
      (let [req-id (::request-id db)]
        (-> db
            (update ::request-id inc)
            (update ::pending-requests conj {:id req-id :submitted-at (::time db)}))))))

(r/defproc ^:fair process-request
  "Process a pending request"
  (fn [db]
    (when (seq (::pending-requests db))
      (let [req (first (::pending-requests db))
            elapsed (- (::time db) (:submitted-at req))]
        (if (< elapsed 5)  ;; Before timeout
          (-> db
              (update ::pending-requests disj req)
              (update ::completed-requests conj (:id req)))
          db)))))  ;; Wait for timeout to handle

(r/defproc ^:fair timeout-request
  "Timeout pending requests that are too old"
  (fn [db]
    (when (seq (::pending-requests db))
      (let [req (first (::pending-requests db))
            elapsed (- (::time db) (:submitted-at req))]
        (when (>= elapsed 5)  ;; After timeout
          (-> db
              (update ::pending-requests disj req)
              (update ::timed-out-requests conj (:id req))))))))

(r/defproc ^:fair advance-time
  "Advance time"
  (fn [db]
    (when (< (::time db) 10)
      (update db ::time inc))))

(r/defproc ^:fair complete
  "Complete when all requests resolved"
  (fn [db]
    (when (and (= (::request-id db) 3)
               (empty? (::pending-requests db)))
      (r/done))))

;; ====================
;; Invariants
;; ====================

(rh/definvariant request-accounted-for
  "Every request is either pending, completed, or timed out"
  [{::keys [request-id pending-requests completed-requests timed-out-requests]}]
  (= request-id
     (+ (count pending-requests)
        (count completed-requests)
        (count timed-out-requests))))

(rh/definvariant no-duplicate-processing
  "Request cannot be both completed and timed out"
  [{::keys [completed-requests timed-out-requests]}]
  (empty? (clojure.set/intersection
           (set completed-requests)
           (set timed-out-requests))))

;; ====================
;; Properties
;; ====================

(rh/defproperty all-requests-eventually-resolved
  "All requests eventually complete or timeout"
  [{::keys [request-id pending-requests]}]
  (rh/eventually
   (and (= request-id 3)
        (empty? pending-requests))))

;; ====================
;; Model Execution
;; ====================

(defn run-request-response-spec []
  @(r/run-model
     initial-state
     #{submit-request
       process-request
       timeout-request
       advance-time
       complete
       request-accounted-for
       no-duplicate-processing
       all-requests-eventually-resolved}
     :trace-example true))

;; Test it
(comment
  (def result (run-request-response-spec))
  (:success? result)

  ;; View request lifecycle
  (doseq [[idx state] (map-indexed vector (:trace result))]
    (println "State" idx ":"
             "Time:" (::time state)
             "Pending:" (count (::pending-requests state))
             "Completed:" (count (::completed-requests state))
             "Timed-out:" (count (::timed-out-requests state))))
  )
```

## Pattern Summary

### Common Patterns Demonstrated

1. **Guards with `when`**: Return `nil` to leave state unchanged
2. **Non-determinism with `r/one-of`**: Explore all possibilities
3. **Process completion with `r/done`**: Avoid deadlock violations
4. **State machine with `case`**: Multi-step processes with local PC
5. **Fairness with `^:fair`**: Required for temporal properties
6. **Constraints**: Prune state space early
7. **Invariants**: Validate safety properties
8. **Properties**: Verify liveness with temporal operators

### Testing Pattern

```clojure
(comment
  ;; Run specification
  (def result (run-spec))

  ;; Check success
  (:success? result)

  ;; View trace
  (count (:trace result))
  (first (:trace result))
  (last (:trace result))

  ;; Inspect specific aspect
  (map ::some-field (:trace result))

  ;; Debug violation
  (when-not (:success? result)
    (println "Error:" (:error result)))
  )
```

## Next Steps

1. Copy an example as a starting point
2. Modify state to match your domain
3. Implement actions as processes
4. Add invariants for safety properties
5. Add properties for liveness
6. Run and iterate!

For more API details, see `api.md`.
