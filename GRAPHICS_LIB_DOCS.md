# Atlas Graphics Library

The Genesis graphics library is a small immediate-mode graphics API shared by:

- Userspace C programs
- Lua programs
- Kernel applications

Include the C API with:

```c
#include <genesis/graphics.h>
```

Widgets must be drawn on every frame. Keep any mutable widget data, such as a
slider value or text buffer, outside the frame loop.

## Window Lifecycle

### `graphics_init_window`

```c
bool graphics_init_window(
    struct graphics_window *window,
    const char *title,
    uint32_t width,
    uint32_t height
);
```

Creates a userspace graphics window.

- `window`: caller-owned window state.
- `title`: window title. A default title is used when this is `NULL`.
- `width`, `height`: requested dimensions. A zero dimension uses the default.
- Returns `true` when the window is ready.

### `graphics_update`

```c
bool graphics_update(struct graphics_window *window);
```

Refreshes the window size, pointer state, button transitions, and queued input
events. Call once at the start of each frame. Returns `false` when the window
has been closed.

### `graphics_update_pointer`

```c
bool graphics_update_pointer(struct graphics_window *window);
```

Updates only the window size and pointer state. This is useful for applications
that poll keyboard input separately.

### `graphics_present`

```c
void graphics_present(struct graphics_window *window);
```

Presents the completed frame. In kernel fullscreen mode it also presents the
mouse pointer.

### `graphics_close`

```c
void graphics_close(struct graphics_window *window);
```

Closes the window and releases its graphics backend state.

## Colors

### `graphics_rgb`

```c
uint32_t graphics_rgb(uint8_t red, uint8_t green, uint8_t blue);
```

Builds a color from 8-bit red, green, and blue components. Colors are stored as
`0xRRGGBB` values.

Built-in colors include:

```c
GRAPHICS_BLACK
GRAPHICS_WHITE
GRAPHICS_GRAY
GRAPHICS_LIGHT_GRAY
GRAPHICS_DARK_GRAY
GRAPHICS_BLUE
GRAPHICS_RED
GRAPHICS_GREEN
```

## Drawing Primitives

All coordinates use the window's top-left corner as `(0, 0)`. Drawing is
clipped to the window surface.

### `graphics_clear`

```c
void graphics_clear(struct graphics_window *window, uint32_t color);
```

Fills the entire window with `color`.

### `graphics_pixel`

```c
void graphics_pixel(
    struct graphics_window *window,
    int32_t x,
    int32_t y,
    uint32_t color
);
```

Draws one pixel.

### `graphics_line`

```c
void graphics_line(
    struct graphics_window *window,
    int32_t x0,
    int32_t y0,
    int32_t x1,
    int32_t y1,
    uint32_t color
);
```

Draws a line from `(x0, y0)` to `(x1, y1)`.

### `graphics_rect`

```c
void graphics_rect(
    struct graphics_window *window,
    int32_t x,
    int32_t y,
    uint32_t width,
    uint32_t height,
    uint32_t color
);
```

Draws a rectangle outline.

### `graphics_fill_rect`

```c
void graphics_fill_rect(
    struct graphics_window *window,
    int32_t x,
    int32_t y,
    uint32_t width,
    uint32_t height,
    uint32_t color
);
```

Draws a filled rectangle.

### `graphics_circle`

```c
void graphics_circle(
    struct graphics_window *window,
    int32_t center_x,
    int32_t center_y,
    uint32_t radius,
    uint32_t color
);
```

Draws a circle outline.

### `graphics_fill_circle`

```c
void graphics_fill_circle(
    struct graphics_window *window,
    int32_t center_x,
    int32_t center_y,
    uint32_t radius,
    uint32_t color
);
```

Draws a filled circle.

### `graphics_text`

```c
void graphics_text(
    struct graphics_window *window,
    int32_t x,
    int32_t y,
    const char *text,
    uint32_t scale,
    uint32_t color
);
```

Draws text using the built-in 5x7 bitmap font. A scale of zero is treated as
one.

### `graphics_blit_scaled`

```c
void graphics_blit_scaled(
    struct graphics_window *window,
    int32_t x,
    int32_t y,
    uint32_t width,
    uint32_t height,
    const uint32_t *pixels,
    uint32_t source_width,
    uint32_t source_height
);
```

Copies an `0xRRGGBB` pixel buffer into the window, scaling it to `width` by
`height`.

## Widgets

Widgets use immediate-mode input. Draw them each frame after
`graphics_update()` and before `graphics_present()`.

### `graphics_button`

```c
bool graphics_button(
    struct graphics_window *window,
    int32_t x,
    int32_t y,
    uint32_t width,
    uint32_t height,
    const char *label
);
```

Draws a beveled button. Returns `true` for the frame in which the left mouse
button is pressed inside the button.

### `graphics_checkbox`

```c
bool graphics_checkbox(
    struct graphics_window *window,
    int32_t x,
    int32_t y,
    uint32_t width,
    uint32_t height,
    bool *checked,
    const char *label
);
```

Draws a checkbox and optional label. Clicking inside its bounds toggles
`*checked`. Returns `true` on the frame where the value toggles.

Example:

```c
bool enabled = false;

while (graphics_update(&window)) {
    graphics_clear(&window, GRAPHICS_BLACK);
    if (graphics_checkbox(&window, 20, 60, 240, 28, &enabled, "Enable feature")) {
        /* enabled changed */
    }
    graphics_present(&window);
}
```

### `graphics_progress`

```c
void graphics_progress(
    struct graphics_window *window,
    int32_t x,
    int32_t y,
    uint32_t width,
    uint32_t height,
    uint32_t value,
    uint32_t maximum
);
```

Draws a non-interactive progress bar. `value` is clamped to `maximum`. A zero
`maximum` draws an empty bar.

### `graphics_slider`

```c
bool graphics_slider(
    struct graphics_window *window,
    int32_t x,
    int32_t y,
    uint32_t width,
    uint32_t height,
    uint32_t *value,
    uint32_t minimum,
    uint32_t maximum
);
```

Draws an interactive horizontal slider and supports click-and-drag input.

- `window`: graphics window being drawn.
- `x`, `y`: top-left corner of the slider bounds.
- `width`, `height`: slider bounds. `width` must be at least `height`.
- `value`: mutable value owned by the caller.
- `minimum`, `maximum`: inclusive value range.
- Returns `true` when dragging changes `*value` during the current frame.

Values outside the range are clamped. If `maximum` is less than `minimum`, the
range is treated as a single value at `minimum`.

Example:

```c
uint32_t volume = 50;

while (graphics_update(&window)) {
    graphics_clear(&window, GRAPHICS_BLACK);
    if (graphics_slider(&window, 20, 60, 300, 28, &volume, 0, 100)) {
        /* volume changed */
    }
    graphics_present(&window);
}
```

### Text Fields

#### `graphics_textbox`

```c
bool graphics_textbox(
    struct graphics_window *window,
    int32_t x,
    int32_t y,
    uint32_t width,
    uint32_t height,
    char *text,
    size_t capacity
);
```

Draws and edits a text field. `text` must be writable and have room for the
terminating null byte. Returns `true` when the text changes.

#### `graphics_text_field`

For persistent text editing state, use:

```c
struct graphics_text_field {
    char *text;
    size_t capacity;
    size_t length;
    size_t cursor;
    bool focused;
    uint64_t state_generation;
};
```

Initialize or synchronize the state with:

```c
void graphics_text_field_init(
    struct graphics_text_field *field,
    char *text,
    size_t capacity
);

void graphics_text_field_sync(
    struct graphics_text_field *field,
    char *text,
    size_t capacity
);
```

The lower-level update function processes queued keyboard events without
drawing:

```c
bool graphics_text_field_update(
    struct graphics_window *window,
    struct graphics_text_field *field,
    int32_t x,
    int32_t y,
    uint32_t width,
    uint32_t height
);
```

It returns `true` when the text buffer changes. Most applications should use
`graphics_textbox_field`, which updates and draws the field together.

Draw and update it with:

```c
bool graphics_textbox_field(
    struct graphics_window *window,
    struct graphics_text_field *field,
    int32_t x,
    int32_t y,
    uint32_t width,
    uint32_t height
);
```

## Input

### `graphics_next_event`

```c
bool graphics_next_event(
    struct graphics_window *window,
    struct genesis_input_event *event
);
```

Reads the next keyboard event queued by `graphics_update()`. It returns `false`
when no events remain.

The window also exposes current pointer state:

```c
window.mouse_x
window.mouse_y
window.left_down
window.right_down
window.left_pressed
window.right_pressed
```

`left_pressed` and `right_pressed` are true only on the transition frame when
the corresponding button changes from up to down.

## Userspace C Example

```c
#include <genesis/graphics.h>

int main(void) {
    struct graphics_window window;
    uint32_t value = 50;

    if (!graphics_init_window(&window, "Slider", 400, 160)) return 1;

    while (graphics_update(&window)) {
        graphics_clear(&window, graphics_rgb(30, 34, 40));
        graphics_text(&window, 20, 20, "Drag the slider", 2, GRAPHICS_WHITE);
        graphics_slider(&window, 20, 70, 340, 28, &value, 0, 100);
        graphics_present(&window);
    }

    graphics_close(&window);
    return 0;
}
```

The same example is available as `.atlas/slider_example.c`.

## Kernel Applications

Kernel applications bind a persistent `graphics_window` to the framebuffer and
input supplied by the app frame callback:

```c
struct app_state {
    struct graphics_window window;
    uint32_t value;
};

static enum app_status frame(
    void *state,
    struct limine_framebuffer *framebuffer,
    const struct app_input *input
) {
    struct app_state *app = state;

    graphics_bind_kernel(&app->window, framebuffer, input, false);
    graphics_clear(&app->window, GRAPHICS_BLACK);
    graphics_slider(&app->window, 20, 60, 300, 28, &app->value, 0, 100);
    return APP_RUNNING;
}
```

Use `true` as the final argument to `graphics_bind_kernel` for a fullscreen
TUI application. Kernel callers must keep the `graphics_window` state between
frames so widget interaction can continue across frames.

## Lua API

The `graphics` module is available automatically in Atlas Lua programs.

### `graphics.init_window`

```lua
local window = graphics.init_window(title, width, height)
```

Creates a window. Width and height are optional.

### Lua Window Methods

| Method | Arguments | Returns |
| --- | --- | --- |
| `update` | none | `open` boolean |
| `present` | none | none |
| `close` | none | none |
| `clear` | `color` | none |
| `pixel` | `x, y, color` | none |
| `line` | `x0, y0, x1, y1, color` | none |
| `rect` | `x, y, width, height, color` | none |
| `fill_rect` | `x, y, width, height, color` | none |
| `circle` | `center_x, center_y, radius, color` | none |
| `fill_circle` | `center_x, center_y, radius, color` | none |
| `text` | `x, y, text, scale, color` | none |
| `button` | `x, y, width, height, label` | clicked boolean |
| `checkbox` | `x, y, width, height, checked, label` | checked, changed boolean |
| `textbox` | `x, y, width, height, text, capacity` | text, changed boolean |
| `progress` | `x, y, width, height, value, maximum` | none |
| `slider` | `x, y, width, height, value, minimum, maximum` | value, changed boolean |
| `size` | none | width, height |
| `mouse` | none | x, y, left-down, right-down |

Lua slider example:

```lua
local window = graphics.init_window("Slider", 400, 160)
local value = 50

while window:update() do
    window:clear(0x000000)
    value = window:slider(20, 70, 340, 28, value, 0, 100)
    window:present()
end

window:close()
```

The same example is available as `.atlas/slider_example.lua`.

### `graphics.rgb`

```lua
local color = graphics.rgb(red, green, blue)
```

Returns an `0xRRGGBB` color integer.

## Frame Pattern

The normal frame order is:

```text
update -> clear -> draw primitives/widgets -> present
```

Do not recreate mutable widget state inside the frame loop. For example, a C
slider value must be declared before the loop, and a Lua slider value must be
assigned from the previous frame's return value.
