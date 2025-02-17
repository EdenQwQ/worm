# Documentation for `wormc`

`wormc` provides the primary way to interact with the Worm window manager's inter-process communication system. It also serves as a reference point for people wanting to make their own IPC clients for Worm.

Here's the basic format that we use for wormc:
```
$ wormc command1 parameters... command2 parameters...
```
A command can have any number of parameters. In theory, all these parameters are not run through the parser, but in practice they currently are; this is an implementation detail which is rather an edge case but still will be fixed soon.

Numbers are always in decimal. Always. There are no exceptions to this rule; colors are also in plain decimal. Eg, this is invalid:
```
$ wormc frame-pixel #00ffff
```
Instead, you could use on posix shells `$((16#00ffff))` and have the shell expand it for you. For readability purposes, this is recommended and will be used in this sheet for examples; in fish, this is `(math 00ffff)`.

Before we begin, finally, a note on X11 colors: We use the 3 byte format RRGGBB in our examples. However, transparency is tacked on to the X11 protocol but it's part of the color. While this is technically implementation dependent because it's not part of the X standard, both Xorg and Xephyr seem to use the format AARRGGBB. If you want control over opacity and are running a compositor use this format, unless you know your X server uses a different one.

Here is the full list of commands:
<br>
### `border-active-pixel(uint)`
Sets the border color for the currently active client. Ex: `wormc border-active-pixel $((16#00ffff))`
### `border-inactive-pixel(uint)`
Sets the border color for all inactive clients. Ex: `wormc border-inactive-pixel $((16#000000))`
### `border-width(uint)`
Sets the border width for all clients, active or not. Ex: `wormc border-width 5`
### `frame-active-pixel(uint)`
Sets the color of the frame (titlebar) for the active window. Ex: `wormc frame-active-pixel $((16#123456))`
### `frame-inactive-pixel(uint)`
Sets the color of the frame (titlebar) for all windows that are inactive. Ex: `wormc frame-inactive-pixel $((16#222222))`
### `frame-height(uint)`
Sets the height of the frame / titlebar for all clients, active or not. Ex: `wormc frame-height 20`
### `text-active-pixel(uint)`
Sets the color of the text drawn on the titlebar / frame for active windows. Ex: `wormc text-pixel $((16#ffffff))`
### `text-inactive-pixel(uint)`
Sets the color of the text drawn on the titlebar / frame for inactive windows. Ex: `wormc text-pixel $((16#000000))`
### `gaps(uint)`
Sets the gaps to the specified amount. When in tiling mode, this distance is reserved between the inside parts of windows. See struts for the outside. Ex: `wormc gaps 5`
### `text-font(string)`
Set the font of text in client titlebars. The provided string must be in valid XFT format. While proper documentation can be found elsewhere, you can do `FontName` or `FontName:size=N`, which covers most use cases; but Xft allows doing much more, like disabling anti-aliasing. Ex `wormc text-font Terminus:size=8`
### `text-offset(uint x, uint y)`
Specifies the offset of text in the titlebar. By default text is positioned at (0,0) which makes it invisible. The Y value needs to be set to something higher, based on the font size. In the future this Y offset will of course be auto-calculated. Ex `wormc text-offset 10 15`
### `kill-client(window id)`
Kills the client with the provided window ID forcefully using XKillClient. Ex `wormc kill-client 1234567890`
### `kill-active-client()`
Same as kill-client, but kills the focused window. Eg `wormc kill-active-client`
### `close-client(window ID)`
Nicely sends a WM_DELETE_WINDOW message from ICCCM to the provided window ID. For example, `wormc close-client 1234567890`
### `close-active-client()`
Same as close-client, but closes the focused window. Eg, `wormc close-active-client`
### `switch-tag(uint in 1-9)`
Clears all other tags and enables given tag. Example: `wormc switch-tag 5`
### `layout(string)`
Changes layout. `floating` for floating, `tiling` for tiling, otherwise wormc exits. Eg `wormc layout tiling`
### `struts(uint top, uint bottom, uint left, uint right)`
Sets the struts, also known as the 'outer margins. These are used when maximizing windows (currently unimplemented, sorry!) and while tiling. Example: `wormc struts 10 50 10 10`.
### `move-tag(uint tag, window id)`
Clears the tags of the provided window and turns on the given tag; for example `wormc move-tag 5 123456789`.
### `move-active-tag(uint tag)`
Like move-tag, but uses the focused window. Eg `wormc move-active-tag 5`
### `master(window id)`
In a tiling layout, sets the master of the current workspace to the given window ID. Ex `wormc master 123456789`
### `master-active()`
Like `master`, but uses the active client. Example: `wormc master-active`
### `float(window id)`
Change the mode of the client indicated by the provided window ID to floating. In a tiling layout, this would indicate that the provided window should not be tiled. TODO: have a way to reverse this affect. As an example: `wormc float 1234567890`
### `float-active()`
Like float, but floats the active client; eg `wormc float-active`.
### `maximize-client(window id)`
Maximize the given window, eg `wormc maximize-client 132123`
### `maximize-active-client()`
`maximize-client`'s equivalent applied on the currently focused/raised/active window. Eg `wormc maximize-active-client`.
### `frame-left(string)`
Describes the layout of the frame *on the left side*. This is a comma separated list of *parts*. The parts can be any of:
 - T for window title
 - C for close button
 - M for maximize button
eg: `wormc frame-left 'T;C;M'
### `frame-center(string)` / `frame-right(string)`
same as frame-left, but for the center and right parts of a frame window.
### `button-size(int)`
The size (both width and height) of all window buttons. The window buttons don't have to nessecarily be perfect squares. You can use the larger dimension and the rest of the window will just not render. Eg: `wormc button-size 14`
### `button-offset(x, y)`
The offset at both the x and y positions at which buttons on the titlebar (M, C) are located, for example `wormc button-offset 10 10`.
### `maximize-path / close-path (string)`
Full path to the maximize and close buttons, respectively. If this is unspecified or the value is invalid, then that part will skip rendering. Eg `wormc maximize-path ~/.config/worm/max.png`.
### `root-menu(string)`
Sets path to the root menu. If this file is valid, upon right-clicking the root window it's executed (assumed to be an executable file). Eg `wormc root-menu ~/worm/examples/jgmenu_run`.
### `decoration-disable(string)`
Disable decorations for all windows which have a class that matches the regex given in `string`. For an example: `wormc decoration-disable '(?i).*firefox.*'`.
Note: The regex format we use is described at [the nim regex docs page](https://nitely.github.io/nim-regex/regex.html)
