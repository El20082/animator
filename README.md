# Animator

**Live app: https://el20082.github.io/animator/**

Animator turns small programs into animated line drawings. You write a `draw`
function in a subset of [Gleam](https://gleam.run), and the app runs it with its
own interpreter and animates the lines it returns, one stroke at a time. Built
for the CS 61A (Fall 2026) Animator project.

This repository holds the published app: `index.html`, `animator.css`, and the
compiled `app.mjs`.

## Using the app

- **Edit the code** on the left. Every change is run right away. If the code
  works, the drawing animates from the start. If it doesn't, a red banner shows
  the error and the line it's on, and the last working drawing stays up.
- **Pick a drawing** from the menu at the top of the code panel, or choose
  **+ New drawing…** and name it. Drawings are saved in your browser, so
  reloading the page keeps them and reopens the one you had open.
- **shape_slider** sets the value passed to `draw`. It runs from 0 to 100 by
  default; type a new maximum in the box at its right end and press Enter.
- **speed** runs from −3× to 3×; negative speeds play the drawing backward.
- **▶ / ⏸** plays or pauses (and replays a finished drawing), and **⏮** restarts
  it.
- **⟨ / ⟩** hides or shows the code panel, and **{ }** shows the raw list of
  paths.

## The drawing language

A drawing is a function `draw(shape_slider: Int) -> List(Path)`, written in a
subset of Gleam: integers and booleans, `let`, `case` with list patterns,
recursion, anonymous functions (`fn(x) { ... }`), function captures, pipes, and
records with field access (`p.x`).

| Built-in | What it is |
| --- | --- |
| `Point(x, y)` | A point. (0, 0) is the centre, x and y run from −500 to 500, and y grows downward. |
| `Line(start, end)` | A straight line between two points. |
| `Path(shape, start, end, thickness)` | Draws a shape between times `start` and `end`, in tenths of a second. |
| `random(min, max)`, `abs(n)` | A random integer in [min, max), and an absolute value. |
| `list.map`, `list.filter`, `list.append`, `list.length`, `list.flatten` | List functions. |

For example, this draws a fan of 30 lines that all appear over three seconds:

```gleam
fn draw(shape_slider: Int) -> List(Path) {
  fan(30, Point(-200, -200))
}

fn fan(n: Int, start: Point) -> List(Path) {
  case n > 0 {
    True -> [Path(Line(start, Point(n * 10, 200)), 0, 30, 3), ..fan(n - 1, start)]
    False -> []
  }
}
```

## Extension: 3D drawings and a call inspector

### 3D drawings

Two new built-ins draw in three dimensions:

| Built-in | What it is |
| --- | --- |
| `Point3(x, y, z)` | A 3D point. x and y are as in 2D, and z runs away from you. |
| `Line3(start, end)` | A straight line between two `Point3`s, used inside a `Path` like any other shape. |

```gleam
Path(Line3(Point3(0, 0, 0), Point3(100, 0, 200)), 0, 10, 3)
```

3D lines are shown in perspective through a camera, and farther lines are drawn
thinner and dimmer. 2D and 3D lines can be mixed in one drawing.

- **Drag** the canvas to turn the camera around the drawing.
- **Scroll** over the canvas, or use **−** and **+**, to zoom out and in.
- **⟲** returns to the front view.

Moving or zooming the camera never restarts the animation. Try the `cube_3d`,
`double_helix_3d`, and `tree_3d` examples. The helix builds its circles from a
12-step cosine table written in the drawing language itself, since the language
only has integers.

### Call inspector

Click **🔍** to see which function call drew each line, a bit like an
environment diagram for the picture.

- **Hover** over a line to turn it white and see the call that drew it. Every
  line drawn by that call (and the calls it made) stays bright, and the rest
  dims.
- **Click** a line to pin it. The panel lists the whole chain of calls it was
  drawn inside, with each call's arguments, such as
  `branch(p: Point(0, 200), d: Point(110, -110), level: 1, depth: 6)`. Click
  any call in the chain to highlight everything that call drew, such as a whole
  branch of the recursive tree.
- The panel also counts the lines each function made, can hide a function's
  lines, and can colour lines by list order, by function, or by call depth.

The inspector works without changing the interpreter. While it's on, the app
wraps each of the program's top-level functions (and `Path`) so that every call
is recorded on a stack and every path is tagged with the calls that were running
when it was made. With the inspector off, programs run exactly as usual.

## Built with

[Gleam](https://gleam.run) compiled to JavaScript, the
[Lustre](https://hexdocs.pm/lustre/) web framework,
[glance](https://hexdocs.pm/glance/) for parsing, and
[CodeMirror](https://codemirror.net) for the code editor.
