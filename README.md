# KitNative — UI Engine in Go ( Plan & Demo )

**Goal:** Bring **web-like declarative UI** to **multi-platform native apps in Go**, with **reactive data binding**, **flexible layout**, and **native rendering**.

---

## 1️⃣ Concept Overview

| Feature               | Description                                                            | Example / Note                                                                          |
| --------------------- | ---------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| Declarative UI        | Write UI in HTML/XML-like syntax                                       | `<flex layout="row"><text bind="username"></text></flex>`                               |
| Reactive Binding      | 3 types: `bind`, `model`, `sync`                                       | `bind="stateVar"` → one-way, `model="stateVar"` → 2-way, `sync="stateVar"` → UI → state |
| Event Handling        | `click`, `submit`, etc., just like web                                 | `<button click="logout()">Log Out</button>`                                             |
| Responsive Layout     | Attributes like `layout:mobile` adapt layout                           | `<flex layout="row" layout:mobile="column">`                                            |
| Scrollable Containers | Vertical/horizontal scroll without extra components                    | `<flex scroll="vertical">...</flex>`                                                    |
| Components            | Cards, buttons, inputs, text                                           | `<card><text>Card 1</text></card>`                                                      |
| Native Rendering      | Render fully native via OpenGL / GLFW / mobile libraries               | No WebView needed                                                                       |
| Multi-platform        | Desktop (Windows/Linux/Mac), Mobile (iOS/Android), future web via WASM | Go-native core handles rendering & input                                                |
| Simplicity            | HTML-like, declarative, familiar for web developers                    | `<button>Click me</button>` vs verbose engine calls                                     |

---

## 2️⃣ UI Example (XML)

```xml
<app title="Demo App">

  <!-- Header -->
  <flex layout="row" layout:mobile="column" padding="16" spacing:bind="headerSpacing" scroll="none">
    <text size="24" weight="bold" color="#222" bind="username"></text>
    <button click="logout()">Log Out</button>
  </flex>

  <!-- Form -->
  <flex layout="column" padding="16" spacing="12" scroll="none">
    <input placeholder="Enter name" sync="username"></input>
    <input placeholder="Enter email" model="email"></input>
    <button click="submitForm()" bind:disabled="!isFormValid">Submit</button>
  </flex>

  <!-- Content Section -->
  <flex layout="row" layout:mobile="column" spacing="8" scroll="vertical">
    <card><text>Card 1</text></card>
    <card><text>Card 2</text></card>
    <card><text>Card 3</text></card>
  </flex>

  <!-- Footer -->
  <flex layout="row" layout:mobile="column" padding="12" spacing="6" scroll="none">
    <button click="openSettings()">Settings</button>
    <button click="openHelp()">Help</button>
  </flex>

</app>
````

---

## 3️⃣ Go Core / Reactive Concept

| Concept         | How it works                                      | Example                                                |
| --------------- | ------------------------------------------------- | ------------------------------------------------------ |
| Node Tree       | Each element (`flex`, `text`, `button`) is a Node | `Node{ID, Props, Children, X,Y,W,H, Texture, OnClick}` |
| Reactive Signal | State variable with subscriptions                 | `username := NewSignal("John")`                        |
| Event Binding   | Nodes store callbacks                             | `buttonNode.OnClick = func(){...}`                     |
| Layout Engine   | Compute positions recursively                     | Row/Column, spacing, padding, scroll offset            |
| Render Engine   | Draw nodes, textures, rectangles, text            | OpenGL / native libraries                              |
| Scroll Handling | Clip viewport, render scrollbars                  | `scrollOffset` calculation                             |
| State Sync      | bind / model / sync                               | `<input sync="username"/>` updates `username` Signal   |

---

## 4️⃣ Planned Features

| Feature                                  | Status    | Notes                                       |
| ---------------------------------------- | --------- | ------------------------------------------- |
| XML/HTML-like UI parser                  | Planned   | Convert XML -> Node tree                    |
| Layout engine (flex + spacing + padding) | Planned   | Support row/column + responsive             |
| Reactive signals                         | Prototype | One-way `bind`, two-way `model`/`sync`      |
| Event system                             | Planned   | `click`, `submit`, etc.                     |
| Scroll containers                        | Prototype | Vertical/horizontal, clipping viewport      |
| Card component                           | Planned   | Reusable UI component                       |
| Input component                          | Planned   | `sync` / `model` bindings                   |
| Multi-platform render                    | Planned   | Desktop (GLFW), Mobile (future iOS/Android) |
| Hot reload XML                           | Future    | Allow live editing UI during development    |
| CSS-like styling                         | Future    | Colors, padding, margins, typography        |

---

## 5️⃣ Goals for AI / Contributors

* Understand **web-like declarative approach** in Go
* Contribute **core engine**: parsing, layout, rendering
* Add **components**: input, card, button, text
* Implement **multi-platform backends**
* Maintain **reactive state system**

---

## 6️⃣ Example controls.go

> Core engine: Node, Signal, Layout, Rendering, Scroll (shortened for demo)

```go
// controls.go
package main
import (
	"fmt"
	"image"
	"image/color"
	"image/draw"
	"io/ioutil"
	"log"
	"runtime"

	"github.com/go-gl/gl/v4.6-core/gl"
	"github.com/golang/freetype"
)
func init(){runtime.LockOSThread()}

// --- Reactive Signal ---
type Signal[T any]{value T; subs []func(T)}
func NewSignal[T any](initial T)*Signal[T]{return &Signal[T]{value:initial}}
func(s*Signal[T])Get()T{return s.value}
func(s*Signal[T])Set(v T){s.value=v; for _,f:=range s.subs{f(v)}}
func(s*Signal[T])Subscribe(f func(T)){s.subs=append(s.subs,f)}

// --- Node ---
type Node struct{ID string; Props map[string]any; Children []*Node; X,Y,W,H float32; Text string; OnClick func(); Texture uint32; Dirty bool}
func NewNode(id string)*Node{return &Node{ID:id,Props:make(map[string]any)}}

// --- Layout Engine ---
func ApplyContainerLayout(node*Node,scrollOffset float32){layout,ok:=node.Props["layout"].(string); if !ok{return}; padding:=float32(5); x,y:=node.X,node.Y-scrollOffset; for _,c:=range node.Children{c.X,c.Y=x,y; if layout=="column"{y+=c.H+padding} else if layout=="row"{x+=c.W+padding}}}

// --- Render Engine ---
func RenderTextToImage(text,fontPath string,w,h int,size float64)*image.RGBA{ /*...*/ }
func NewTextureFromImage(img*image.RGBA)uint32{ /*...*/ }
func DrawTexture(x,y,w,h float32,tex uint32){ /*...*/ }
func drawRect(x,y,w,h float32,col [3]float32){ /*...*/ }

// --- Scrollbar ---
func DrawScrollbar(x,y,w,h,scrollOffset,contentHeight,viewportHeight float32){ /*...*/ }

// --- Demo template ---
func ParseTemplate()(*Node,*Node){ /*...*/ }
```

---

## 7️⃣ How to Run Demo

```bash
go get github.com/go-gl/gl/v4.6-core/gl
go get github.com/go-glfw/glfw/v3.3/glfw
go get github.com/golang/freetype
go run main.go
```

You will see a **scrollable container** with 30 lines and a **reactive "+1" button**.

---

## 8️⃣ How to Contribute / Extend Kit Native

### 8.1 Core Engine

* **Parser**: Implement XML/HTML parsing to build Node trees.
* **Layout**: Extend flex engine with `justify`, `align`, `wrap`, or media queries.
* **Render**: Support more backends (OpenGL, Metal, Vulkan, iOS UIKit, Android Jetpack Compose).

### 8.2 Components

* Add new **UI elements**: sliders, checkboxes, dropdowns, images, charts.
* Build **reusable components** with props and event bindings.
* Support **nested components** and dynamic children.

### 8.3 Reactive System

* Add more **binding types**, e.g., computed, derived, or async.
* Optimize updates for **partial re-rendering**.
* Support **nested signals** or **list rendering**.

### 8.4 Multi-platform

* Desktop: GLFW, SDL2, OpenGL, Vulkan.
* Mobile: iOS/Android native rendering.
* WASM/Web: optional web backend with `canvas` rendering.

### 8.5 Tools & Dev Experience

* Hot reload XML or Go templates.
* CSS-like styling system.
* Dev server with live UI preview.
* Example apps and pre-built templates.

### 8.6 Pull Request Guidelines

1. Fork the repo and create a branch.
2. Implement feature or fix bug.
3. Write **example usage** in `demo` folder.
4. Add tests where possible.
5. Open PR with description, screenshots, and rationale.

---

✅ This README is **both documentation and vision**:

* Explains **concepts, reactive design, and layout engine**.
* Provides **demo code** to get started.
* Guides **contributors** on extending Kit Native engine and components.

