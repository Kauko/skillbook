# Structurizr DSL Styling Reference

Complete guide to themes, styles, colors, shapes, and visual customization in Structurizr DSL.

## Overview

Styling in Structurizr DSL controls the visual appearance of diagrams through element styles, relationship styles, themes, and branding. Styles are applied based on element/relationship tags, enabling consistent visualization across multiple views.

## Element Styles

Element styles define the visual appearance of diagram elements (people, systems, containers, components).

### Syntax

```dsl
styles {
    element <tag> {
        shape <shape>
        icon <file|url>
        width <integer>
        height <integer>
        background <color>
        color <color>
        stroke <color>
        strokeWidth <integer: 1-10>
        fontSize <integer>
        border <solid|dashed|dotted>
        opacity <integer: 0-100>
        metadata <true|false>
        description <true|false>
    }
}
```

### Shape Options

**Available shapes:**
- `Box` - Rectangle (default)
- `RoundedBox` - Rectangle with rounded corners
- `Circle` - Circle
- `Ellipse` - Ellipse/oval
- `Hexagon` - Hexagon
- `Cylinder` - Cylinder (common for databases)
- `Pipe` - Pipe shape
- `Person` - Person icon/shape
- `Robot` - Robot icon
- `Folder` - Folder icon
- `WebBrowser` - Browser window frame
- `MobileDevicePortrait` - Vertical mobile device
- `MobileDeviceLandscape` - Horizontal mobile device
- `Component` - Component box

**Example:**
```dsl
styles {
    element "Person" {
        shape person
    }
    element "Database" {
        shape cylinder
    }
    element "External" {
        shape hexagon
    }
    element "Mobile" {
        shape MobileDevicePortrait
    }
}
```

### Colors

Colors can be specified as hex codes or CSS/HTML color names:

```dsl
element "System" {
    background #1168BD        # Hex code
    color #ffffff             # Hex code
    stroke DarkBlue          # CSS color name
}

element "Component" {
    background SteelBlue     # CSS color name
    color white              # CSS color name
}
```

**Common color names:**
- `white`, `black`, `gray`, `silver`
- `red`, `blue`, `green`, `yellow`, `orange`, `purple`, `pink`
- `navy`, `teal`, `olive`, `maroon`, `lime`, `aqua`
- `DarkBlue`, `SteelBlue`, `LightGray`, etc.

### Visual Properties

```dsl
element "Critical" {
    # Border
    stroke #ff0000
    strokeWidth 5
    border solid              # solid, dashed, dotted

    # Transparency
    opacity 90                # 0-100 (0=fully transparent)

    # Typography
    fontSize 24               # Font size in points

    # Dimensions
    width 200                 # Width in pixels
    height 150                # Height in pixels
}
```

### Content Control

Control display of metadata and descriptions:

```dsl
element "Simple" {
    metadata false           # Hide metadata
    description true         # Show description
}
```

### Icons

Add custom icons to elements:

```dsl
element "AWS" {
    icon aws-logo.png
}

element "Service" {
    icon https://example.com/icons/service.svg
}
```

### Complete Example

```dsl
styles {
    element "Person" {
        shape person
        background #08427B
        color #ffffff
        fontSize 22
    }

    element "Software System" {
        shape RoundedBox
        background #1168BD
        color #ffffff
        strokeWidth 2
        stroke #084276
    }

    element "Container" {
        background #438DD5
        color #ffffff
        shape RoundedBox
    }

    element "Component" {
        background #85BBF0
        color #000000
        fontSize 14
    }

    element "Database" {
        shape cylinder
        background #438DD5
        color #ffffff
    }

    element "External" {
        background #999999
        color #ffffff
        border dashed
        opacity 70
    }

    element "Critical" {
        stroke #ff0000
        strokeWidth 4
        border solid
    }

    element "Microservice" {
        shape hexagon
        background #91C4F2
        color #000000
    }

    element "Mobile" {
        shape MobileDevicePortrait
        background #438DD5
        color #ffffff
    }

    element "WebApp" {
        shape WebBrowser
        background #85BBF0
        color #000000
    }
}
```

## Relationship Styles

Relationship styles control the appearance of connections between elements.

### Syntax

```dsl
styles {
    relationship <tag> {
        thickness <integer>
        color <color>
        style <solid|dashed|dotted>
        routing <Direct|Orthogonal|Curved>
        fontSize <integer>
        width <integer>
        position <integer: 0-100>
        opacity <integer: 0-100>
    }
}
```

### Line Styles

```dsl
relationship "Synchronous" {
    style solid
    thickness 2
    color #000000
}

relationship "Asynchronous" {
    style dashed
    thickness 2
    color #666666
}

relationship "Optional" {
    style dotted
    thickness 1
    color #999999
}
```

### Routing Modes

Control how relationship lines are drawn:

```dsl
relationship "Direct" {
    routing Direct           # Straight lines
}

relationship "Flow" {
    routing Orthogonal       # Right-angle lines
}

relationship "Curved" {
    routing Curved           # Curved/spline lines
}
```

**Routing options:**
- `Direct`: Straight lines between elements
- `Orthogonal`: Right-angle (Manhattan) routing
- `Curved`: Smooth curved lines

### Label Control

```dsl
relationship "API" {
    fontSize 14              # Label font size
    position 50              # Label position along line (0-100)
    color #000000           # Label and line color
}

relationship "Internal" {
    position 30              # Position near source
}

relationship "External" {
    position 70              # Position near destination
}
```

**Position:** 0 (source) to 100 (destination), 50 (middle)

### Complete Example

```dsl
styles {
    relationship "Relationship" {
        thickness 2
        color #707070
        style solid
        routing Direct
        fontSize 14
    }

    relationship "Synchronous" {
        style solid
        thickness 3
        color #000000
    }

    relationship "Asynchronous" {
        style dashed
        thickness 2
        color #666666
    }

    relationship "External" {
        style dotted
        thickness 1
        color #999999
        opacity 60
    }

    relationship "Critical" {
        thickness 4
        color #ff0000
        style solid
    }

    relationship "DataFlow" {
        routing Curved
        thickness 2
        color #438DD5
        position 60
    }
}
```

## Themes

Themes provide pre-defined styling that can be reused across workspaces.

### Syntax

```dsl
# Single theme
theme <url|file|default>

# Multiple themes
themes <url|file> [url|file] ... [url|file]
```

### Default Theme

The `default` keyword references the official Structurizr theme:

```dsl
views {
    styles {
        # Your custom styles
    }
    theme default
}
```

### Custom Themes

Reference themes via URL or file:

```dsl
# Remote theme
theme https://example.com/themes/corporate.json

# Local theme file
theme themes/corporate-theme.json

# Multiple themes (later themes override earlier ones)
themes https://structurizr.com/themes/default themes/corporate.json
```

### Theme Loading

"Themes specified via a URL will be loaded dynamically when required" or can reference local files that are inlined into the workspace.

### Theme Files

Themes are JSON files defining element and relationship styles:

```json
{
  "name": "Corporate Theme",
  "description": "Company branding",
  "elements": [
    {
      "tag": "Person",
      "shape": "Person",
      "background": "#08427B",
      "color": "#ffffff"
    },
    {
      "tag": "Container",
      "background": "#438DD5",
      "color": "#ffffff"
    }
  ],
  "relationships": [
    {
      "tag": "Relationship",
      "thickness": 2,
      "color": "#707070"
    }
  ]
}
```

### Theme Precedence

Styles are applied in this order (later overrides earlier):
1. Default theme (if specified)
2. Custom themes (in order specified)
3. Inline styles in workspace DSL

## Branding

Customize rendering output across diagrams and documentation.

### Syntax

```dsl
views {
    branding {
        logo <file|url>
        font <name> [url]
    }
}
```

### Logo

Add a company logo to diagrams:

```dsl
branding {
    logo corporate-logo.png
}

branding {
    logo https://example.com/branding/logo.svg
}
```

### Custom Fonts

Specify custom fonts:

```dsl
branding {
    font "Open Sans"
}

branding {
    font "Roboto" "https://fonts.googleapis.com/css2?family=Roboto"
}
```

### Complete Example

```dsl
views {
    branding {
        logo images/company-logo.png
        font "Inter" "https://fonts.googleapis.com/css2?family=Inter"
    }
}
```

## Complete Styling Example

```dsl
workspace "Styled Architecture" {
    model {
        user = person "User" "System user" {
            tags "Person"
        }

        system = softwareSystem "Main System" {
            tags "Internal"

            webapp = container "Web App" "SPA" "React" {
                tags "WebApp"
            }

            api = container "API" "REST API" "Node.js" {
                tags "Backend" "Critical"
            }

            database = container "Database" "Data store" "PostgreSQL" {
                tags "Database" "Critical"
            }
        }

        external = softwareSystem "External Service" {
            tags "External"
        }

        user -> webapp "Uses" "HTTPS" {
            tags "External"
        }
        webapp -> api "Calls" "REST" {
            tags "Synchronous" "Critical"
        }
        api -> database "Reads/writes" "SQL" {
            tags "Synchronous" "Critical"
        }
        api -> external "Integrates" "REST" {
            tags "Asynchronous" "External"
        }
    }

    views {
        systemContext system "Context" {
            include *
            autoLayout
        }

        container system "Containers" {
            include *
            autoLayout
        }

        styles {
            # Element styles
            element "Person" {
                shape person
                background #08427B
                color #ffffff
                fontSize 22
            }

            element "Software System" {
                background #1168BD
                color #ffffff
                shape RoundedBox
            }

            element "Container" {
                background #438DD5
                color #ffffff
                shape RoundedBox
            }

            element "Database" {
                shape cylinder
                background #438DD5
                color #ffffff
            }

            element "External" {
                background #999999
                color #ffffff
                border dashed
                opacity 70
            }

            element "Critical" {
                stroke #ff0000
                strokeWidth 4
            }

            element "WebApp" {
                shape WebBrowser
            }

            element "Backend" {
                shape hexagon
            }

            # Relationship styles
            relationship "Synchronous" {
                style solid
                thickness 2
                color #000000
            }

            relationship "Asynchronous" {
                style dashed
                thickness 2
                color #666666
            }

            relationship "External" {
                style dotted
                thickness 1
                color #999999
            }

            relationship "Critical" {
                thickness 4
                color #ff0000
            }
        }

        branding {
            logo images/logo.png
            font "Inter" "https://fonts.googleapis.com/css2?family=Inter"
        }

        theme default
    }
}
```

## C4 Model Default Colors

Standard C4 model color scheme:

```dsl
element "Person" {
    background #08427B
    color #ffffff
}

element "Software System" {
    background #1168BD
    color #ffffff
}

element "Container" {
    background #438DD5
    color #ffffff
}

element "Component" {
    background #85BBF0
    color #000000
}
```

## Compatibility Notes

"Element styles are designed to work with the Structurizr cloud service/on-premises installation/Lite, and may not be fully supported by the PlantUML, Mermaid, etc export formats."

**Export format limitations:**
- Some shapes not supported in PlantUML/Mermaid
- Opacity may not work in all formats
- Custom fonts require format support
- Icons may render differently or not at all
- Relationship routing modes vary by format

## Best Practices

1. **Use tags consistently** - Apply tags at element/relationship creation
2. **Start with default theme** - Override only what needs customization
3. **Create theme files** - Reuse styles across workspaces
4. **Test exports** - Verify styles work in target format (PNG, SVG, PlantUML, etc.)
5. **Limit custom colors** - Stick to a defined palette
6. **Use meaningful shapes** - Person for users, cylinder for databases, etc.
7. **Apply visual hierarchy** - Use color, size, and stroke to show importance
8. **Consider accessibility** - Ensure sufficient color contrast
9. **Document theme choices** - Explain color/shape meanings
10. **Version control themes** - Track theme files in source control

## Color Palettes

### Standard C4 Colors
```dsl
# Blue scale
#08427B  # Dark blue (Person)
#1168BD  # Medium blue (System)
#438DD5  # Light blue (Container)
#85BBF0  # Very light blue (Component)
```

### Extended Palette
```dsl
# Status colors
#2ECC40  # Green (success, active)
#FF851B  # Orange (warning, pending)
#FF4136  # Red (error, critical)
#B10DC9  # Purple (planned, future)

# Neutrals
#111111  # Black (text)
#AAAAAA  # Gray (disabled, deprecated)
#DDDDDD  # Light gray (borders)
#FFFFFF  # White (background)
```

### Technology Colors
```dsl
element "AWS" { background #FF9900 }        # AWS Orange
element "Azure" { background #0078D4 }      # Azure Blue
element "GCP" { background #4285F4 }        # Google Blue
element "Kubernetes" { background #326CE5 } # K8s Blue
element "Docker" { background #2496ED }     # Docker Blue
element "React" { background #61DAFB }      # React Cyan
element "Node" { background #339933 }       # Node Green
element "Python" { background #3776AB }     # Python Blue
element "Java" { background #007396 }       # Java Blue
element "Go" { background #00ADD8 }         # Go Cyan
```

## Visual Design Patterns

### Layered Architecture
```dsl
element "Presentation" { background #85BBF0 }  # Light blue
element "Application" { background #438DD5 }   # Medium blue
element "Domain" { background #1168BD }        # Dark blue
element "Infrastructure" { background #08427B } # Darkest blue
```

### Microservices
```dsl
element "Service" {
    shape hexagon
    background #438DD5
    color #ffffff
}

element "Gateway" {
    shape RoundedBox
    background #1168BD
    strokeWidth 3
}
```

### External vs Internal
```dsl
element "Internal" {
    background #1168BD
    border solid
}

element "External" {
    background #999999
    border dashed
    opacity 70
}
```
