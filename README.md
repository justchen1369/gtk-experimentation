
---> DLE EOT n
---> [Name]
Real-time status transmission
---> [Format]
ASCII DLE EOT n
Hex 10 04 n
Decimal 16 4 n
---> [Range]
1 ≤ n ≤ 4
---> [Description]
Transmits the printer status specified by ‘n’ in real-time.
---> [Note]
The printer will return relevant status immediately after receiving this command.
---> [Reference]
None


## 2 Command Description ........................................................................................................................

### HT

[Name]
Horizontal tab
[Format]
ASCII HT
Hex 09
Decimal 9
[Description]
Moves the print position to the next horizontal tab position.
[Notes]
• This command is ignored unless the next horizontal tab position has been set.

- If the next horizontal tab position exceeds the printing area, the printer sets the printing
    position to [Printing area width + 1].
- Horizontal tab positions are set with ESC D.
- If this command is received when the printing position is at [printing area width + 1], the
    printer executes print buffer-full printing of the current line and horizontal tab processing
    from the beginning of the next line.
- The default setting of the horizontal tab position for the paper roll is font A(12×24) every
    8th character.
[Reference]
ESC D

### LF

[Name]
Print and line feed
[Format]
ASCII LF
Hex 0A
Decimal 10
[Description]
Prints the data in the print buffer and feeds one line based on the current line spacing.
[Note]
This command sets the print position to the beginning of the line.
[Reference]
ESC 2, ESC 3

### FF

[Name]
Print and return to standard mode in page mode.
[Format]
ASCII FF
Hex 0C
Decimal 12
[Description]
Prints the data in the print buffer collectively and returns to standard mode in page mode,
prints the data in the print buffer and feeds one line based on the current line spacing in
standard mode.
[Notes]
· This command is valid only in page mode.
· The buffer data is deleted after being printed.
· The printer does not execute paper cutting.
· This command sets the print position to the beginning of the line.
[Reference]
ESC FF, ESC L, ESC S


### CR

[Name]
Print and carriage return
[Format]
ASCII CR
Hex 0D
Decimal 13
[Description]
When automatic line feed is enabled, this command functions the same as LF; when
automatic line feed is disabled, this command is ignored.
[Notes]
· Sets the print starting position to the beginning of the line.
· This command is set according to the printer configuration.
[Reference]
LF

### CAN

[Name]
Cancel print data in page mode
[Format]
ASCII CAN
Hex 18
Decimal 24
[Description]
In page mode, deletes all the print data in the current area.
[Notes]
· This command is enabled only in page mode.
· If data that existed in the previously specified printing area also exists in the currently
specified printing area, it is deleted.
[Reference]
ESC L, ESC W

### DLE EOT N

[Name]
Real-time status transmission
[Format]
ASCII DLE EOT n
Hex 10 04 n
Decimal 16 4 n
[Range]
1 ≤ n ≤ 4
[Description]
Transmits the selected printer status specified by n in real-time, according to the following
parameters:
n = 1: Transmit printer status
n = 2: Transmit off-line status
n = 3: Transmit error status
n = 4: Transmit paper roll sensor status
[Notes]
· The status is transmitted whenever the data sequence of <10>H<04>H< n>
(1 ≤ n ≤ 4) is received.
Example:
In ESC * m nL nH d1...dk, d1=<10>H, d2=<04>H, d3=<01>H
· This command should not be used within the data sequence of another command that
consists of 2 or more bytes.


```
Example:
If you attempt to transmit ESC 3 n to the printer, but DTR (DSR for the host computer)
goes to MARK before n is transmitted and then DLE EOT 3 interrupts before n is received,
the code <10>H for DLE EOT 3 is processed as the code for ESC 3 <10>H.
· Even though the printer is not selected using ESC = (select peripheral device), this
command is effective.
· The printer transmits the current status. Each status is represented by one-byte data.
· The printer transmits the status without confirming whether the host computer can receive
data.
· The printer executes this command upon receiving it.
· This command is executed even when the printer is off-line, the receive buffer is full, or
there is an error status with a serial interface model.
· With a parallel interface model, this command is ignored.
· When Auto Status Back (ASB) is enabled using the GS a command, the status
transmitted by the DLE EOT command and the ASB status must be differentiated.
```
n = 1: Printer status

```
Bit Off/On Hex Decimal Function
0 Off 00 0 Not used. Fixed to Off
1 On 02 2 Not used. Fixed to On
2 Off 00 0 Drawer open/close signal is LOW (connector pin 3)
On 04 4 Drawer open/close signal is HIGH (connector pin 3)
3 Off 00 0 On-line.
On 08 8 Off-line
4 On 10 16 Not used. Fixed to On
5,6 Undefined
7 Off 00 00 Not used. Fixed to Off.
```
n = 2: Off-line status

```
Bit Off/On Hex Decimal Function
0 Off 00 0 Not used. Fixed to Off
1 On 02 2 Not used. Fixed to On
2 Off 00 0 Cover is closed.
On 04 4 Cover is open
3 Off 00 0 Paper is not being fed by using the FEED button
On 08 8 Paper is being fed by the FEED button
4 On 10 16 Not used. Fixed to On
5 Off 00 0 No paper-end stop.
On 20 32 Printing is being stopped.
6 Off 00 0 No error.
On 40 64 Error occurs
7 Off 00 0 Not used. Fixed to Off
```

Bit 5: Becomes on when the paper end sensor detects paper end and printing.
n = 3: Error status
Bit Off/On Hex Decimal Function
0 Off 00 0 Not used. Fixed to Off
1 On 02 2 Not used. Fixed to On
2 Undefined
3 Off^00 0 No auto-cutter error
On 08 8 Auto-cutter error occurs.
4 On 10 16 Not used. Fixed to On
5 Off^00 0 No unrecoverable error
On 20 32 Unrecoverable error occurs
6 Off^00 0 No auto-recoverable error.
On 40 64 Auto recoverable error occurs
7 Off 00 0 Not used. Fixed to Off

Bit 3: If these errors occur due to paper jams or the like, it is possible to recover by correcting the cause
of the error and executing DLE ENQ n (1 ≤ n ≤ 2). If an error due to a circuit failure (e.g. wire break)
occurs, it is impossible to recover.
Bit 6: When printing is stopped due to high print head temperature until the print head temperature drops
sufficiently or when the paper roll cover is open during printing, bit 6 is On.
n = 4: Continuous paper sensor status
Bit Off/On Hex Decimal Function
0 Off 00 0 Not used. Fixed to Off
1 On 02 2 Not used. Fixed to On
2,3 Off^00 0 Paper roll near-end sensor: paper adequate
On 0C 12 Paper near-end is detected by the paper roll near-end
4 On 10 16 Not used. Fixed to On.
5,6 Off^00 0 Paper roll sensor: Paper present
On 60 96 Paper roll end detected by paper roll sensor.
7 Off 00 0 Not used. Fixed to Off
[Reference]
DLE ENQ, GS a, GS r

### DLE ENQ N

[Name]
Real-time request to printer
[Format]
ASCII DLE ENQ n
Hex 10 05 n
Decimal 16 5 n
[Range]
1 ≤n ≤ 2
[Description]
Responds to a request from the host computer, n specifies the requests as follows:

```
n Request
1 Recover from an error and restart printing from the line where the error occurred
2 Recover from an error after clearing the receive and print buffers
```

[Notes]
· This command is effective only when an auto-cutter error occurs.
· The printer starts processing data upon receiving this command.
· This command is executed even when the printer is off-line, the receive buffer is full, or
there is an error status with a serial interface model.
· With a parallel interface model, this command can not be executed when the printer is
busy.
· The status is also transmitted whenever the data sequence of <10>H<05>H< n>
(1 ≤n ≤ 2) is received.
Example:
In ESC * m nL nH dk, d1 = <10>H, d2 = <05>H, d3 = <01>H
· This command should not be contained within another command that consists of two or
more bytes.
Example:
If you attempt to transmit ESC 3 n to the printer, but DTR (DSR for the host computer)
goes to MARK before n is transmitted, and DLE ENQ 2 interrupts before n is received,
the code <10>H for DLE ENQ 2 is processed as the code for ESC 3 <10>H.
· DLE ENQ 2 enables the printer to recover from an error after clearing the data in the
receive buffer and the print buffer. The printer retains the settings (by ESC !, ESC 3, etc.)
that were in effect when the error occurred. The printer can be initialized completely by
using this command and ESC @. This command is enabled only for errors that have the
possibility of recovery, except for print head temperature error.
· When the printer is disabled with ESC = (Select peripheral device), the error recovery
functions (DLE ENQ 1 and DLE ENQ 2) are enabled, and the other functions are
disabled.
[Reference]
DLE EOT

### DLE DC4 N M T

[Name]
Generate pulse at real-time
[Format]
ASCII DLE DC4 n m t
Hex 10 14 n m t
Decimal 16 20 n m t
[Range]
n = 1
m = 0, 1
1 ≤ t≤ 6
[Description]
Outputs the pulse specified by t to connector pin m as follows:

```
m Connector pin
0 Drawer kick-out connector pin 2
1 Drawer kick-out connector pin 5
The pulse ON time is [ t × 100 ms] and the OFF time is [ t × 100ms].
```

[Notes]
When the pulse is output to the connector pin specified while ESC p or DEL DC4 is
executed while this command is processed, this command is ignored.
The printer executes this command upon receiving it.· With a serial interface model, this
command is executed even when the printer is off-line, the receive buffer is full, or there is
an error status.
· With a parallel interface model, this command cannot be executed when the printer is
busy.
· If print data includes the same character strings as this command, the printer performs
the same operation specified by this command. The user must consider this.
· This command should not be used within the data sequence of another command that
consists of 2 or more bytes.
· This command is effective even when the printer is disabled with ESC = (Select
peripheral device).
[Reference]
ESC p

### ESC FF

[Name]
Print data in page mode
[Format]
ASCII ESC FF
Hex 1B 0C
Decimal 27 12
[Description]
In page mode, prints all buffered data in the printing area collectively.
[Notes]
· This command is enabled only in page mode.
· After printing, the printer does not clear the buffered data, setting values for ESC T and
ESC W, and the position for buffering character data.
[Reference]
FF, ESC L, ESC S

### ESC SP N

[Name]
Set right-side character spacing
[Format]
ASCII ESC SP n
Hex 1B 20 n
Decimal 27 32 n
[Range]
0 ≤ n ≤ 255
[Description]
Sets the character spacing for the right side of the character to [ n × horizontal or vertical
motion units].
[Notes]
The right-side character spacing for double-width mode is twice the normal value. When
characters are enlarged, the right-side character spacing is n times normal value.
·This command sets values independently in each mode (standard and page modes).
·The horizontal and vertical motion unit are specified by GS P. Changing the horizontal or
vertical motion unit does not affect the current right-side spacing.
·In standard mode, the horizontal motion unit is used.


·In page mode, the horizontal or vertical motion unit differs in page mode, depending on
starting position of the printable area as follows:
1 When the starting position is set to the upper left or lower right of the printable area
using ESC T, the horizontal motion unit ( x) is used.
2 When the starting position is set to the upper right or lower left of the printable area
using ESC T, the vertical motion unit ( y) is used.
·The maximum right-side spacing is 255/180 inches. Any setting exceeding the maximum
is converted to the maximum automatically.
[Default] n = 0
[Reference]
GS P

### ESC! N

[Name]
Select print mode(s)
[Format]
ASCII ESC! n
Hex 1B 21 n
Decimal 27 33 n
[Range]
0 ≤ n ≤ 255
[Description]
Selects print mode(s) using n as follows:
Bit Off/On Hex Decimal Function
0 Off^00 0 Character font A (12 × 24)
On 01 1 Character font B (9 × 17)
1,2 Undefined.
3 Off^00 0 Emphasized mode not selected
On 08 8 Emphasized mode selected
4 Off^00 0 Double-height mode not selected
On 10 16 Double-height mode selected
5 Off^00 0 Double-width mode not selected
On 20 32 Double-width mode selected
6 Undefined
7 Off^00 0 Underline mode not selected
On 80 128 Underline mode selected

[Notes]
· When both double-height and double-width modes are selected, quadruple size
characters are printed.
· The printer can underline all characters, but can not underline the space set by HT or 90°
clockwise rotated characters.
· The thickness of the underline is that selected by ESC -, regardless of the character size.
· When some characters in a line are double or more height, all the characters on the line
are aligned at the baseline.
· ESC E can also turn on or off emphasized mode. However, the setting of the last received
command is effective.
· ESC - can also turn on or off underline mode. However, the setting of the last received


command is effective.
· GS! can also select character size. However, the setting of the last received command is
effective.
[Default] n = 0
[Reference]
ESC -, ESC E, GS!

### ESC $ NL NH

[Name]
Set absolute print position
[Format]
ASCII ESC $ nL nH
Hex 1B 24 nL nH
Decimal 27 36 nL nH
[Range]
0 ≤nL ≤ 255
0 ≤nH≤ 255
[Description]
Sets the distance from the beginning of the line to the position at which subsequent
characters are to be printed.
· The distance from the beginning of the line to the print position is [( nL + nH × 256) ×
(vertical or horizontal motion unit)] inches.
[Notes]
· Settings outside the specified printable area are ignored.
· The horizontal and vertical motion unit are specified by GS P.
· In standard mode, the horizontal motion unit ( x) is used.
· In page mode, horizontal or vertical motion unit differs depending on the starting position
of the printable area as follows:
1 When the starting position is set to the upper left or lower right of the printable area
using ESC T, the horizontal motion unit ( x) is used.
2 When the starting position is set to the upper right or lower left of the printable area
using ESC T, the vertical motion unit ( y) is used.
[Reference]
ESC \, GS $, GS \, GS P

### ESC % N

[Name]
Select/cancel user-defined character set
[Format]
ASCII ESC % n
Hex 1B 25 n
Decimal 27 37 n
[Range]
0 ≤ n ≤ 255
[Description]
Selects or cancels the user-defined character set.
· When the LSB of n is 0, the user-defined character set is canceled.
· When the LSB of n is 1, the user-defined character set is selected.
[Notes]
· When the user-defined character set is canceled, the internal character set is
automatically selected.
· n is available only for the least significant bit.


[Default] n = 0
[Reference]
ESC &, ESC?

### ESC & Y C 1 C 2 [X 1 D1D(Y × X1)][XK D1D(Y × XK)]

[Name]
Define user-defined characters
[Format]
ASCII ESC & y c1 c2 [x1 d1...d(y × x1)]...[xk d1...d(y × xk)]
Hex 1B 26 y c1 c2 [x1 d1...d(y × x1)]...[xk d1...d(y × xk)]
Decimal 27 38 y c1 c2 [x1 d1...d(y × x1)]...[xk d1...d(y × xk)]
[Range]
y = 3
32 ≤ c1 ≤ c2 ≤ 127
0< x ≤ 24
0 ≤ d1 ... d(y × xk) ≤ 255
[Description]
Defines user-defined characters.
· y specifies the number of bytes in the vertical direction.
· c1 specifies the beginning character code for the definition, and c2 specifies the final
code.
· x specifies the number of dots in the horizontal direction.
[Notes]
· The allowable character code range is from ASCII code <20>H to <7F>H (96 characters).
· It is possible to define multiple characters for consecutive character codes. If only one
character is desired, use c1 = c2.
· d is the dot data for the characters. The dot pattern is in the horizontal direction from the
left side. Any remaining dots on the right side are blank.
· The data to define a user-defined character is (y × x) bytes.
· Set a corresponding bit to 1 to print a dot or 0 to not print a dot.
· A user-defined character and a downloaded bit image can be defined simultaneously.
· The user-defined character definition is cleared when:
①ESC? is executed.
②The the power is turned off.
· When the user-defined characters are defined in font B (9 × 17), only the most significant
bit of the 3rd byte of data in vertical direction is effective.
[Default] The internal character set
[Reference]
ESC %, ESC?
[Example] · When font A (12 × 24) is selected.


```
· When font B (9×17) is selected.
```
### ESC * M NL NH D1 DK

[Name]
Select bit-image mode
[Format]
ASCII ESC * m nL nH d1...dk
Hex 1B 2A m nL nH d1...dk
Decimal 27 42 m nL nH d1...dk


[Range]
m = 0, 1, 32, 33
0 ≤ nL ≤ 255
0 ≤ nH ≤ 3
0 ≤ d ≤ 255
[Description]
Selects a bit-image mode using m for the number of dots specified by nL and nH,
as follows:
m Mode Vertical Direction Horizontal Direction
Number of Dots Dot Density Dot density Number of Data (K)
0 8-dot single-density 8 60 DPI 101 DPI nL + nH × 256
1 8-dot double-density 8 60 DPI 203 DPI nL + nH × 256
32 24-dot single-density 24 180 DPI 101 DPI ( nL + nH × 256) × 3
33 24-dot double-density 24 180 DP 203 DPI ( nL + nH × 256) × 3

[Notes]
If the values of m is out of the specified range, nL and data following are processed as
normal data.
The nL and nH indicate the number of dots of the bit image in the horizontal direction. The
number of dots is calculated by nL + nH × 256.
If the bit-image data input exceeds the number of dots to be printed on a line, the excess
data is ignored.
d indicates the bit-image data. Set a corresponding bit to 1 to print a dot or to 0 to not print a
dot.
After printing a bit image, the printer returns to normal data processing mode.
·This command is not affected by print modes (emphasized, double-strike, underline,
character size or white/black reverse printing), except upside-down printing mode.
· The relationship between the image data and the dots to be printed is as follows:
· When 8-dot bit image is selected:


```
When 24-dot bit image is selected:
```
### ESC - N

[Name]
Turn underline mode on/off
[Format]
ASCII ESC - n
Hex 1B 2D n
Decimal 27 45 n
[Range]
0 ≤ n ≤2, 48 ≤n ≤ 50
[Description]
Turns underline mode on or off, based on the following values of n:
n Function
0, 48 Turns off underline mode
1, 49 Turns on underline mode (1-dot thick)
2, 50 Turns on underline mode (2-dots thick)

[Notes]
The printer can underline all characters (including right-side character spacing), but cannot
underline the space set by HT.
The printer cannot underline 90° clockwise rotated characters and white/black inverted
characters.
· When underline mode is turned off by setting the value of n to 0 or 48, the following data
is not underlined, and the underline thickness set before the mode is turned off does not
change. The default underline thickness is 1 dot.
Changing the character size does not affect the current underline thickness.
Underline mode can also be turned on or off by using ESC !. Note, however, that the last
received command is effective.
[Default] n = 0
[Reference]
ESC!


### ESC 2

[Name]
Select default line spacing
[Format]
ASCII ESC 2
Hex 1B 32
Decimal 27 50
[Description]
Selects 1/6-inch lines spacing (approximately 4.23mm).
[Notes]
· The line spacing can be set independently in standard mode and in page mode.
[Reference]
ESC 3

### ESC 3 N

[Name]
Set line spacing
[Format]
ASCII ESC 3 n
Hex 1B 33 n
Decimal 27 51 n
[Range]
0 ≤ n ≤ 255
[Description]
Sets the line spacing to [ n × vertical or horizontal motion unit] inches.
[Notes]
·The line spacing can be set independently in standard mode and in page mode.
·The horizontal and vertical motion unit are specified by GS P. Changing the horizontal or
vertical motion unit does not affect the current line spacing.
· In standard mode, the vertical motion unit (y) is used.
· In page mode, this command functions as follows, depending on the starting position of
the printable area:
① When the starting position is set to the upper left or lower right of the printable area
using ESC T, the vertical motion unit (y) is used.
② When the starting position is set to the upper right or lower left of the print able area
using ESC T, the horizontal motion unit ( x) is used.
· The maximum paper feed amount is 1016 mm (40 inches). Even if a paper feed amount
of more than 1016 mm (40 inches) is set, the printer feeds the paper only 1016 mm (
inches).
[Default] Line spacing equivalent to approximately 4.23mm (1/6 inches).
[Reference]
ESC 2, GS P

### ESC = N

[Name]
Set peripheral device
[Format]
ASCII ESC = n
Hex 1B 3D n
Decimal 27 61 n
[Range]
0 ≤ n ≤ 1


[Description]
Selects device to which host computer sends data, using n as follows:
Bit Off/On Hex Decimal Function
0 Off^00             0 Printer disabled
On 01 1 Printer enabled
1-7 Undefined

[Notes]
· When the printer is disabled, it ignores all data except for error-recovery commands (DLE
EOT, DLE ENQ, DLE DC4) until it is enabled by this command.
[Default] n = 1

### ESC? N

[Name]
Cancel user-defined characters
[Format]
ASCII ESC? n
Hex 1B 3F n
Decimal 27 63 n
[Range]
32 ≤n ≤ 127
[Description]
Cancels user-defined characters.
[Notes]
·This command cancels the pattern defined for the character code specified by n. After the
user-defined characters is canceled, the corresponding pattern for the internal character is
printed.
·This command deletes the pattern defined for the specified code in the font selected by
ESC !.
· If a user-defined character has not been defined for the specified character code, the
printer ignores this command.
[Reference]
ESC &, ESC %

### ESC @

[Name]
Initialize printer
[Format]
ASCII ESC @
Hex 1B 40
Decimal 27 64
[Description]
Clears the data in the print buffer and resets the printer mode to the mode that was in
effect when the power was turned on.
[Notes]
· The DIP switch settings are not checked again.
· The data in the receive buffer is not cleared.
· The macro definition is not cleared.
· The NV bit image data is not cleared.
· The data of the user NV memory is not cleared.


### ESC D N1NK NUL

[Name]
Set horizontal tab positions
[Format]
ASCII ESC D n1... nk NUL
Hex 1B 44 n1...nk 00
Decimal 27 68 n1...nk 0
[Range]
1 ≤ n ≤ 255
1 ≤ k ≤ 32
[Description]
Sets horizontal tab positions.
·n specifies the column number for setting a horizontal tab position from the beginning of
the line.
· k indicates the total number of horizontal tab positions to be set.
[Notes]
·The horizontal tab position is stored as a value of [character width × n] measured from the
beginning of the line. The character width includes the right-side character spacing, and
double-width characters are set with twice the width of normal characters.
· This command cancels the previous horizontal tab settings.
·When setting n = 8, the print position is moved to column 9 by sending HT.
·Up to 32 tab positions ( k = 32) can be set. Data exceeding 32 tab positions is processed
as normal data.
·Transmit [ n] k in ascending order and place a NUL code 0 at the end.
·When [ n] k is less than or equal to the preceding value [ n] k-1, tab setting is finished and
the following data is processed as normal data.
·ESC D NUL cancels all horizontal tab positions.
·The previously specified horizontal tab positions do not change, even if the character
width changes.
·The character width is memorized for each standard and page mode.
[Default] The default tab positions are at intervals of 8 characters (columns 9, 17, 25,...) for font A
(12 × 24).
[Reference]
HT

### ESC E N

[Name]
Turn emphasized mode on/off
[Format]
ASCII ESC E n
Hex 1B 45 n
Decimal 27 69 n
[Range]
0 ≤ n ≤ 255
[Description]
Turns emphasized mode on or off
When the LSB of n is 0, emphasized mode is turned off.
When the LSB of n is 1, emphasized mode is turned on.


[Notes]
·Only the least significant bit of n is enabled.
·This command and ESC! turn on and off emphasized mode in the same way. Be careful
when this command is used with ESC !.
[Default] n = 0
[Reference]
ESC!

### ESC G N

[Name]
Turn on/off double-strike mode
[Format]
ASCII ESC G n
Hex 1B 47 n
Decimal 27 71 n
[Range]
0 ≤ n ≤ 255
[Description]
Turns double-strike mode on or off.
·When the LSB of n is 0, double-strike mode is turned off.
·When the LSB of n is 1, double-strike mode is turned on.
[Notes]
·Only the lowest bit of n is enabled.
·Printer output is the same in double-strike mode and in emphasized mode.
[Default] n = 0
[Reference]
ESC E

### ESC J N

[Name]
Print and feed paper
[Format]
ASCII ESC J n
Hex 1B 4A n
Decimal 27 74 n
[Range]
0 ≤n ≤ 255
[Description]
Prints the data in the print buffer and feeds the paper [ n × vertical or horizontal motion
unit] inches.
[Notes]
·After printing is completed, this command sets the print starting position to the beginning
of the line.
·The paper feed amount set by this command does not affect the values set by ESC 2 or
ESC 3.
·The horizontal and vertical motion unit are specified by GS P.
·In standard mode, the printer uses the vertical motion unit ( y).
·In page mode, this command functions as follows, depending on the starting position of
the printable area:
① When the starting position is set to the upper left or lower right of the printable area
using ESC T, the vertical motion unit (y) is used.
② When the starting position is set to the upper right or lower left of the print able area
using ESC T, the horizontal motion unit ( x) is used.


·The maximum line spacing is 1016mm (40 inches). When the setting value exceeds the
maximum, it is converted to the maximum automatically.
[Reference]
GS P

### ESC L

[Name]
Select page mode
[Format]
ASCII ESC L
Hex 1B 4C
Decimal 27 76
[Description]
Switches from standard mode to page mode.
[Notes]
·This command is enabled only when processed at the beginning of a line in standard
mode.
·This command has no effect in page mode.
·After printing by FF is completed or by using ESC S, the printer returns to standard mode.
·This command sets the position where data is buffered to the position specified by ESC T
within the printing area defined by ESC W.
·This command switches the settings for the following commands (in which the values can
be set independently in standard mode and page mode) to those for page mode:
①Set right-side character spacing: ESC SP, FS S
②Select default line spacing: ESC 2, ESC 3
· Only value settings is possible for the following commands in page mode; these
commands are not executed.
①Turn 90° clockwise rotation mode on/off: ESC V
②Select justification: ESC a
③Turn upside-down printing mode on/off: ESC {
④Set left margin: GS L
⑤Set printable area width: GS W
·The printer returns to standard mode when power is turned on, the printer is reset, or ESC
@ is used.
[Reference]
FF, CAN, ESC FF, ESC S, ESC T, ESC W, GS $, GS \

### ESC M N

[Name]
Select character font
[Format]
ASCII ESC M n
Hex 1B 4D n
Decimal 27 77 n
[Range]
n = 0, 1, 2,3,48, 49,50,51


[Description]
Selects character fonts.
n Function
0,48 Character font A (12 × 24) selected
1,49 Character font B (9 × 17) selected
2,50 User defined character selected
3,51 Chinese font(24 × 24) selected

### ESC R N

[Name]
Select an international character set
[Format]
ASCII ESC R n
Hex 1B 52 n
Decimal 27 82 n
[Range]
0 ≤ n ≤ 12
[Description]
Selects an international character set n from the following table:
n Character set n Character set
0 U.S.A. 7 Spain I
1 France 8 Japan
2 Germany 9 Norway
3 U.K. 10 Denmark II
4 Denmark I 11 Spain II
5 Sweden 12 Latin America
6 Italy 13 Korea
[Default] n = 0

### ESC S

[Name]
Select standard mode
[Format]
ASCII ESC S
Hex 1B 53
Decimal 27 83
[Description]
Switches from page mode to standard mode.
[Notes]
· This command is effective only in page mode.
· Data buffered in page mode are cleared.
· This command sets the print position to the beginning of the line.
· The printing area set by ESC W are initialized.
· This command switches the settings for the following commands (in which the values can
be set independently in standard mode and page mode) to those for standard mode:
①Set right-side character spacing: ESC SP, FS S
②Select default line spacing: ESC 2, ESC 3
· The following commands are enabled only to set in standard mode.
①Set printing area in page mode: ESC W
②Select print direction in page mode: ESC T


· The following commands are ignored in standard mode.
①Set absolute vertical print position in page mode: GS $
②Set relative vertical print position in page mode: GS \
· Standard mode is selected automatically when power is turned on, the printer is reset, or
command ESC @ is used.
[Reference]
FF, ESC FF, ESC L

### ESC T N

[Name]
Select print direction in page mode
[Format]
ASCII ESC T n
Hex 1B 54 n
Decimal 27 84 n
[Range]
0 ≤ n ≤ 3
48 ≤ n ≤ 51
[Description]
Selects the print direction and starting position in page mode.
n specifies the print direction and starting position as follows:

[Notes]
· When the command is input in standard mode, the printer executes only internal flag
operation. This command does not affect printing in standard mode.
· This command sets the position where data is buffered within the printing area set by
ESC W.
· Parameters for horizontal or vertical motion units ( x or y) differ as follows, depending on
the starting position of the printing area:
①If the starting position is the upper left or lower right of the printing area, data is buffered
in the direction perpendicular to the paper feed direction:
Commands using horizontal motion units: ESC SP, ESC $, ESC \
Commands using vertical motion units: ESC 3, ESC J, GS $, GS \
②If the starting position is the upper right or lower left of the printing area, data is buffered
in the paper feed direction:
Commands using horizontal motion units: ESC 3, ESC J, GS $, GS \
Commands using vertical motion units: ESC SP, ESC $, ESC \
[Default] n = 0
[Reference]
ESC $, ESC L, ESC W, ESC \, GS $, GS P, GS \


### ESC V N

[Name]
Turn 90° clockwise rotation mode on/off
[Format]
ASCII ESC V n
Hex 1B 56 n
Decimal 27 86 n
[Range]
0 ≤ n ≤ 1, 48 ≤ n ≤ 49
[Description]
Turns 90° clockwise rotation mode on/off
n is used as follows:
n Function
0, 48 Turns off 90° clockwise rotation mode
1, 49 Turns on 90° clockwise rotation mode

[Notes]
· This command affects printing in standard mode. However, the setting is always effective.
· When underline mode is turned on, the printer does not underline 90° clockwise-rotated.
· Double-width and double-height commands in 90° rotation mode enlarge characters in
the opposite directions from double-height and double- width commands in normal mode.
[Default] n = 0
[Reference]
ESC !, ESC -

### ESC W XL XH YL YH DXL DXH DYL DYH

[Name]
Set printing area in page mode
[Format]
ASCII ESC W xL xH yL yH dxL dxH dyL dyH
Hex 1B 57 xL xH yL yH dxL dxH dyL dyH
Decimal 27 87 xL xH yL yH dxL dxH dyL dyH
[Range]
0 ≤ xL, xH, yL, yH, dxL, dxH, dyL, dyH ≤ 255 (except dxL= dxH=0 or dyL= dyH=0)
[Description]
· The horizontal starting position, vertical starting position, printing area width, and printing
area height are defined as x0, y0, dx (inch), dy (inch), respectively.
Each setting for the printing area is calculated as follows:
x0 = [( xL + xH × 256) × (horizontal motion unit)]
y0 = [( yL + yH × 256) × (vertical motion unit)]
dx = [ dxL + dxH × 256) × (horizontal motion unit)]
dy = [ dyL + dyH × 256) × (vertical motion unit)]
The printing area is set as shown in the figure below.
[Notes]
· If this command is input in standard mode, the printer executes only internal flag
operation. This command does not affect printing in standard mode.
· If the horizontal or vertical starting position is set outside the printable area, the printer
stops command processing and processes the following data as normal data.
· If the printing area width or height is set to 0, the printer stops command processing and
processes the following data as normal data.
· This command sets the position where data is buffered to the position specified by ESC T
within the printing area.


```
· If (horizontal starting position + printing area width) exceeds the printable area, the
printing area width is automatically set to (horizontal printable area horizontal starting
position).
· If (vertical starting position + printing area height) exceeds the printable area, the printing
area height is automatically set to (vertical printable area – vertical starting position).
· The horizontal and vertical motion unit are specified by GS P. Changing the horizontal or
vertical motion unit does not affect the current printing area.
· Use the horizontal motion unit ( x) for setting the horizontal starting position and printing
area width, and use the vertical motion unit ( y) for setting the vertical starting position and
printing area height.
· When the horizontal starting position , vertical starting position, printing area width, and
printing area height are defined as X, Y, Dx, and Dy respectively, the printing area is set
as shown in the figure below.
```
[Default] xL = xH = yL = yH = 0
dxL, dxH, dyL, dyH is decided by printer configuration
[Reference]
CAN, ESC L, ESC T, GS P

### ESC \ NL NH

[Name]
Set relative print position
[Format]
ASCII ESC \ nL nH
Hex 1B 5C nL nH
Decimal 27 92 nL nH
[Range]
0 ≤ nL ≤ 255
0 ≤ nH ≤ 255
[Description]
Sets the print starting position based on the current position by using the horizontal or
vertical motion unit.
· This command sets the distance from the current position to [( nL + nH × 256) × horizontal
or vertical motion unit]
[Notes]
· Any setting that exceeds the printable area is ignored.
· When pitch N is specified to the right: , nL+ nH × 256 = N


When pitch N is specified to the left (the negative direction), use the complement of 65536.
When pitch N is specified to the left: nL+ nH × 256 = 65536 - N
· The print starting position moves from the current position to [ N × horizontal or vertical
motion unit]
· The horizontal and vertical motion unit are specified by GS P.
· In standard mode, the horizontal motion unit is used.
· In page mode, the horizontal or vertical motion unit differs as follows,depending on the
starting point of the printing area:
①When the starting position is set to the upper left or lower right of the printable area
using ESC T, the horizontal motion unit ( x) is used.
②When the starting position is set to the upper right or lower left of the printable area
using ESC T, the vertical motion unit ( y) is used.
[Reference]
ESC $, GS P

### ESC A N

[Name]
Select justification
[Format]
ASCII ESC a n
Hex 1B 61 n
Decimal 27 97 n
[Range]
0 ≤ n ≤ 2, 48 ≤ n ≤ 50
[Description]
Aligns all the data in one line to the specified position
n selects the justification as follows:
n Justification
0,48 Left justification
1, 49 Centering
2, 50 Right justification

[Notes]
· The command is enabled only when processed at the beginning of the line in standard
mode.
· If this command is input in page mode, the printer performs only internal flag operations.
· This command has no effect in page mode.
· This command executes justification in the printing area.
· This command justifies the space area according to HT, ESC $ or ESC \.
[Default] n = 0
[Example]


### ESC C 0 N

[Name]
Select the paper type
[Format]
ASCII ESC c 0 n
Hex 1B 63 30 n
Decimal 27 99 48 n
[Range]
n = 0 or 0x99
[Description]
Select the paper type
n = 0, set paper type as normal paper roll.
n = 0x99, set paper type as Marked paper.
[Notes]
·Marked paper is the paper with black mark.
[Default] n = 0
[Reference]
GS FF

### ESC C 3 N

[Name]
Select paper sensor(s) to output paper end signals
[Format]
ASCII ESC c 3 n
Hex 1B 63 33 n
Decimal 27 99 51 n
[Range]
0 ≤ n ≤ 255
[Description]
Selects the paper sensor(s) to output paper end signals
· Each bit of n is used as follows:
Bit Off/On Hex Decimal Function
0 Off^00 0 Paper roll near-end sensor disabled
On 01 1 Paper roll near-end sensor enabled
1 Off^00 0 Paper roll near-end sensor disabled
On 02 2 Paper roll near-end sensor enabled
2-7 Undefined

[Notes]
· It is possible to select multiple sensors to output signals. Then, if any of the sensors
detects a paper end, the paper end signal is output.
· The command is available only with a parallel interface and is ignored with a serial
interface.
· Sensor is switched when executing this command. The paper end signal switching be
delayed depending on the receive buffer state.
· If either bit 0 or bit 1 is on, the paper roll near-end sensor is selected as the paper sensor
outputting paper-end signals
· When all the sensors are disabled, the paper end signal always outputs a paper present
status.
[Default] n = 15


### ESC C 4 N

[Name]
Select paper sensor(s) to stop printing
[Format]
ASCII ESC c 4 n
Hex 1B 63 34 n
Decimal 27 99 52 n
[Range]
0 ≤ n ≤ 255
[Description]
Selects the paper sensor(s) used to stop printing when a paper-end is detected,
using n as follows:
Bit Off/On Hex Decimal Function
0 Off^00 0 Paper roll near end sensor disabled
On 01 1 Paper roll near end sensor enabled
1 Off^00 0 Paper roll near end sensor disabled
On 02 2 Paper roll near end sensor enabled
2-7 Undefined

[Notes]
· When a paper sensor is enabled with this command, printing is stopped only when the
corresponding paper is selected for printing.
· When a paper-end is detected by the paper roll sensor, the printer goes off-line after
printing stops.
· When either bit 0 or 1 is on, the printer selects the paper roll near-end sensor for the
paper sensor to stop printing.
[Default] n = 0

### ESC C 5 N

[Name]
Enable/disable panel buttons
[Format]
ASCII ESC c 5 n
Hex 1B 63 35 n
Decimal 27 99 53 n
[Range]
0 ≤ n ≤ 255
[Description]
Enables or disables the panel buttons.
· When the LSB of n is 0, the panel buttons are enabled.
· When the LSB of n is 1, the panel buttons are disabled.
[Notes]
· Only the lowest bit of n is valid.
· When the panel buttons are disabled, none of them are usable when the printer cover is
closed.
· In this printer, the panel buttons are the FEED button.
· In the macro ready mode, the FEED button is enabled regardless of the settings of this
command; however, the paper cannot be fed by using these buttons.
[Default] n = 0


### ESC D N

[Name]
Print and feed n lines
[Format]
ASCII ESC d n
Hex 1B 64 n
Decimal 27 100 n
[Range]
0 ≤n ≤ 255
[Description]
Prints the data in the print buffer and feeds n lines.
[Notes]
· This command sets the print starting position to the beginning of the line.
· This command does not affect the line spacing set by ESC 2 or ESC 3.
· The maximum paper feed amount is 1016 mm (40 inches). If the paper feed amount ( n x
line spacing) of more than 1016 mm (40 inches) is specified, the printer feeds the paper
only 1016 mm (40 inches).
[Reference]
ESC 2, ESC 3

### ESC P M T 1 T2

[Name]
Generate pulse
[Format]
ASCII ESC p m t1 t2
Hex 1B 70 m t1 t2
Decimal 27 112 m t1 t2
[Range]
m = 0, 1, 48, 49
0 ≤ t1 ≤ 255, 0≤ t2 ≤ 255
[Description]
Outputs the pulse specified by t1 and t2 to connector pin m as follows:
m Connector pin
0, 48 Drawer kick-out connector pin 2
1, 49 Drawer kick-out connector pin 5

[Notes]
· The pulse ON time is [ t1 × 2 ms] and the OFF time is [ t2 × 2 ms].
· If t2 < t1, the OFF time is [ t1 × 2 ms]
[Reference]
DLE DC 4

### ESC T N

[Name]
Select character code table
[Format]
ASCII ESC t n
Hex 1B 74 n
Decimal 27 116 n
[Range]
0 ≤ n ≤ 7, 16 ≤ n ≤ 19


[Description]
Selects a page n from the character code table.
n Page
0 PC437 [U.S.A., Standard Europe]
1 Katakana
2 PC850 [Multilingual]
3 PC860 [Portuguese]
4 PC863 [Canadian-French]
5 PC865 [Nordic]
16 WPC1252
17 PC866 [Cyrillic #2]
18 PC852 [Latin2]
19 PC858
255 Space Page

[Default] n = 0

### ESC { N

[Name]
Turns on/off upside-down printing mode
[Format]
ASCII ESC { n
Hex 1B 7B n
Decimal 27 123 n
[Range]
0 ≤ n ≤ 255
[Description]
Turns upside-down printing mode on or off.
· When the LSB of n is 0, upside-down printing mode is turned off.
· When the LSB of n is 1, upside-down printing mode is turned on.
[Notes]
· Only the lowest bit of n is valid.
· This command is enabled only when processed at the beginning of a line in standard
mode.
· When this command is input in page mode, the printer performs only internal flag
operations.
· This command does not affect printing in page mode.
· In upside-down printing mode, the printer rotates the line to be printed by 180° and then
prints it.
[Default] n = 0
[Example]


### FS G 1 M A 1 A 2 A 3 A 4 NL NH D1DK

[Name]
Write to user NV memory
[Format]
ASCII FS g 1 m a1 a2 a3 a4 nL nH d1...dk
Hex 1C 67 31 m a1 a2 a3 a4 nL nH d1...dk
Decimal 28 103 49 m a1 a2 a3 a4 nL nH d1...dk
[Range]
m = 0
0 ≤ ( a1+( a2×256)+( a3×65536)+( a4×16777216)) ≤ 1023
1 ≤ ( nL+( nH×256)) ≤ 1024
32 ≤ d ≤ 255
k = ( nL+( nH×256))
[Description]
Writes data to user NV memory.
· m is always set to 0.
· a1, a2, a3, and a4 specify the data stored starting address to ( a1+( a2×256)
×( a3×65536)+( a4×16777216)).
· nL, nH select the number of stored data bytes ( nL+( nH×256)).
· d specifies the stored data.
[Notes]
· Frequent write command execution by FS g 1 may damage the NV memory.
Therefore, it is recommended to write the NV memory 10 times or less a day.
· User NV memory means the memory area which is used for storing character font data in
non-volatile memory.
· This command is available only when processed at the beginning of a line in standard
mode.
· This command is ignored in page mode.
· When this command is received during macro definition, the printer ends macro definition
and begins executing this command.
· If the values of the argument ( m), the stored starting address ( a1, a2, a3, a4), and the
number of the stored data ( nL, nH) are out of the specified range, or if the stored starting
address ( a1, a2, a3, a4) + the number of the stored data ( nL, nH) ≥ 1024, this command
is ignored and data following are processed as normal data.
· If the value of the stored data d is out of range, the execution of this command is ended,
and data following are processed as normal data. In this case, the data which are stored
in the NV memory still remain.
· Writing data to the NV memory overwrites previous data. Therefore, previous data is
deleted.
· Data which are stored in the user NV memory can be read by FS g 2.
· Once data is stored in the user NV memory, it is not erased by executing ESC @, FS q,
reset, or power off.


### FS G 2 M A 1 A 2 A 3 A 4 NL NH

[Name]
Read from user NV memory
[Format]
ASCII FS g 2 m a1 a2 a3 a4 nL nH
Hex 1C 67 32 m a1 a2 a3 a4 nL nH
Decimal 28 103 50 m a1 a2 a3 a4 nL nH
[Range]
m = 0
0 ≤ ( a1+( a2×256)+( a3×65536)+( a4×16777216)) ≤ 1023
1 ≤ ( nL+( nH×256)) ≤ 80
[Description]
Transmits data from user NV memory.
· m is always set to 0.
· a1, a2, a3, and a4 specify the data stored starting address to
( a1+( a2×256) ×( a3×65536)+( a4×16777216)).
· nL, nH select the number of stored data bytes ( nL+( nH×256)).
[Notes]
The printer transmits all data collectively without confirming whether the host is
ready to receive data. To receive all data result correctly, (the capacity of the transmitted
data + 2) bytes or more space is required in the receive buffer.
· During data transmission, the printer ignores real-time commands. Also, the printer does
not transmit ASB even when the ABS is enabled. Therefore, the user cannot confirm
changes in the printer status during these periods.
· User NV memory means the memory area which is used for storing character font data in
non-volatile memory.
· If the values of the argument ( m), the stored starting address ( a1, a2, a3, a4) and the
number of the stored data ( nL, nH) are out of the specified range, or if the stored staring
address ( a1, a2, a3, a4) + the number of the stored data ( nL, nH) ≥1024, this command is
ignored and data following are processed as normal data.
· After the data is ready to be transmitted, the printer executes the following process:
transmits [Header + Data + NUL].
· The contents of [Header + Data + NUL] are as follows:
Header: Hexadecimal = 5FH / Decimal = 95 (1 byte)
Data: Data stored in user NV memory (( nL+( nH×256)) bytes)
NUL: Hexadecimal = 00H / Decimal = 0 (1 byte)
· the printer transmits all data consecutively without confirming whether the host computer
is ready to receive data. The data transmission must be consecutive, except for the XOFF
code.
· Data which is stored in the user NV memory can be written by FS g 1.


### FS P N M

[Name]
Print NV bit image
[Format]
ASCII FS p n m
Hex 1C 70 n m
Decimal 28 112 n m
[Range]
1 ≤ n ≤ 255
0 ≤ m ≤ 3 , 48 ≤ m ≤ 51
[Description]
Prints a NV bit image n using the mode specified by m.
m Mode Vertical Dot Density (DPI) Horizontal Dot Density (DPI)
0.48 Normal 180 203
1.49 Double-width 180 101
2.50 Double-height 90 203
3.51 Quadruple 90 101
· n is the number of the NV bit image (defined using the FS q command).
· m specifies the bit image mode.
[Detail] · NV bit image means a bit image which is defined in a non-volatile memory by FS q and
printed by FS p.
· This command is not effective when the specified NV bit image has not been defined.
· In standard mode, this command is effective only when there is no data in the print buffer.
· This command is not affected by print modes (emphasized, double-strike, underline,
character size, white/black reverse printing, or 90° rotated characters, etc.), except
upside-down printing mode.
· If the downloaded bit-image to be printed exceeds one line, the excess data is not printed.
· This command feeds dots (for the height n of the NV bit-image) in normal and
double-width modes, and (for the height n × 2 of the NV bit-image) in double-height and
quadruple modes, regardless of the line spacing specified by ESC 2 or ESC 3.
· After printing the bit image, this command sets the print position to the beginning of the
line and processes the data that follows as normal data.
[References] ESC *, FS q, GS /, GS v 0

### FS Q N [XL XH YL YH D1DK]1[XL XH YL YH D1DK]N

[Name]
Define NV bit image
[Format]
ASCII FS q n [ xL xH yL yH d1...dk]1...[ xL xH yL yH d1...dk]n
Hex 1C 71 n [xL xH yL yH d1...dk]1...[ xL xH yL yH d1...dk]n
Decimal 28 113 n [xL xH yL yH d1...dk]1...[ xL xH yL yH d1...dk]n
[Range]
1 ≤ n ≤ 255
0 ≤ xL ≤ 255
0 ≤ xH ≤ 3 (when 1 ≤ ( xL + xH × 256) ≤ 1023
0 ≤ yL ≤ 1 (when 1 ≤ ( yL + yH × 256) ≤ 288
0 ≤ d ≤ 255


k = ( xL + xH × 256) × ( yL + yH × 256) × 8
Total defined data area = 1M bits (128K bytes)
[Description]
Define the NV bit image specified by n.
· n specifies the number of the defined NV bit image.
· xL, xH specifies ( xL + xH × 256) × 8 dots in the horizontal direction for the NV bit image
you are defining.
· yL, yH specifies ( yL + yH × 256) × 8 dots in the vertical direction for the NV bit image you
are defining.
[Notes]
Frequent write command execution may cause damage the NV memory. Therefore, it is
recommended to write the NV memory 10 times or less a day.
·This command cancels all NV bit images that have already been defined by this command.
The printer can not redefine only one of several data definitions previously defined. In this
case, all data needs to be sent again.
· Before the ending of the processing of this command mechanical operations (including
initializing the position of the printer head when the cover is open, paper feeding by
using the FEED button, etc.)cannot be performed. NV bit image means a bit image which
is defined in a non-volatile memory by FS q and printed by FS p.
· In standard mode, this command is effective only when processed at the beginning of the
line.
· This command is effective when 7 bytes <FS~yH> is processed as a normal value.
· When the amount of data exceeds the capacity left in the range defined by xL, xH, yL, yH,
the printer processes xL, xH, yL, yH out of the defined range.
· In the first group of NV bit images, when any of the parameters xL, xH, yL, yH is out of the
definition range, this command is disabled.
· In groups of NV bit images other than the first one, when the printer processes xL, xH, yL,
yH out of the defined range, it stops processing this command and starts writing into the
NV images. At this time, NV bit images that haven’t been defined are disabled (undefined),
but any NV bit images before that are enabled.
· The d indicates the definition data. In data ( d) a 1 bit specifies a dot to be printed and a 0
bit specifies a dot not to be printed.
· This command defines n as the number of a NV bit image. Numbers rise in order from NV
bit image 01H. Therefore, the first data group [xL xH yL yH d1...dk] is NV bit image 01H,
and the last data group [xL xH yL yH d1...dk] is NV bit image n. The total agrees with the
number of NV bit images specified by command FS p.
· A definition data of a NV bit image consists of [xL xH yL yH d1...dk]. Therefore, when only
one NV bit image is defined n=1, the printer processes a data group [xL xH yL yH d1...dk]
once. The printer uses ([data: ( xL + xH × 256) × ( yL + yH × 256) × 8] + [header:4]) bytes
of NV memory.
· The definition area in this printer is a maximum of 1M bits (128K bytes). This command
can define several NV bit images, but cannot define a bit image data whose total capacity


[bit image data + header] exceeds 1M bytes (128K bytes).
· The printer is busy immediately before writing into NV memory.
· When this command is received during macro definition, the printer ends macro definition,
and begins performing this command.
· Once a NV bit image is defined, it is not erased by performing ESC @, reset, and power
off.
· This command performs only definition of a NV bit image and does not perform printing.
Printing of the NV bit image is performed by the FS p command.
[Reference]
FS p
[Example] When xL = 64, xH = 0, yL = 96, yH = 0

### GS FF

[Name]
Feed label to print position
[Format]
ASCII GS FF
Hex 1D 0C
Decimal 29 12
[Description]
Feed label to print position.
[Notes]. Feed paper until the next mark get to special position.
· This command is valid only when the paper type is set to marked paper.
Never use continuous paper when paper type is set to marked paper, otherwise GS FF
command will cause the printer feeding too long.
Never use marked paper when paper type is set to continuous paper, otherwise printer will
get error paper state.
[Reference]
ESC c 0


### GS! N

[Name]
Select character size
[Format]
ASCII GS! n
Hex 1D 21 n
Decimal 29 33 n
[Range]
0 ≤ n ≤ 255
(1 ≤ vertical number of times ≤ 6, 1 ≤ horizontal number of times ≤ 6)
[Description]
Selects the character height using bits 0 to 2 and selects the character width using
bits 4 to 7, as follows:
Bit Off/On Hex Decimal Function
0-3 Character height selection. See Table 2.
4-7 Character width selection. See Table 1.

(^) Table 1 Character Width Selection
Hex Decimal Width
00 0 1 (normal)
10 16 2(double-width)
20 32 3
30 48 4
40 64 5
50 80 6
Table 2 Character Height Selection
Hex Decimal Width
00 0 1 (normal)
01 1 2 (double-height)
02 2 3
03 3 4
04 4 5
05 5 6
[Notes]
This command is all characters (alphanumeric and Kanji) effective except for HRI
characters.
If n is outside of the defined range, this command is ignored.
In standard mode, the vertical direction is the paper feed direction, and the horizontal
direction is perpendicular to the paper feed direction. However,
when character orientation changes in 90° clockwise-rotation mode, the relationship
between vertical and horizontal directions is reversed.
In page mode, vertical and horizontal directions are based on the character orientation.
When characters are enlarged with different sizes on one line, all the characters on the line
are aligned at the baseline.
The ESC! command can also turn double-width and double-height modes on or off.
However, the setting of the last received command is effective.


[Default] n = 0
[Reference]
ESC!

### GS $ NL NH

[Name]
Set absolute vertical print position in page mode
[Format]
ASCII GS $ nL nH
Hex 1D 24 nL nH
Decimal 29 36 nL nH
[Range]
0 ≤ nL ≤ 255, 0 ≤ nH ≤ 255
[Description]
·Sets the absolute vertical print starting position for buffer character data in page mode.
This command sets the absolute print position to [( nL + nH × 256) × (vertical or horizontal
motion unit)] inches.
[Notes]
This command is effective only in page mode.
· If the [( nL + nH × 256) × (vertical or horizontal motion unit)] exceeds the specified
printing area, this command is ignored.
· The horizontal starting buffer position does not move.
· The reference starting position is that specified by ESC T.
· This command operates as follows, depending on the starting position of the printing
area specified by ESC T:
① When the starting position is set to the upper left or lower right, this command sets the
absolute position in the vertical direction.
② When the starting position is set to the upper right or lower left, this command sets the
absolute position in the horizontal direction.
· The horizontal and vertical motion unit are specified by GS P.
[Reference]
ESC $, ESC T, ESC W, ESC \, GS P, GS \, 3.12 Page Mode

### GS ( A PL PH N M

[Name]
Execute test print
[Format]
ASCII GS ( A pL pH n m
Hex 1D 28 41 pL pH n m
Decimal 29 40 65 pL pH n m
[Range]
( pL+( pH × 56))=2 (where pL=2, pH=0)
0 ≤ n ≤ 2, 8 ≤ n ≤ 50
1 ≤ m≤ 3, 9 ≤ m ≤ 51


[Description]
· Executes a test print with a specified test pattern on a specified paper. pL and pH
specifies the number of the parameter such as n, m to [ pL + ( pH × 256)] bytes.
n specifies the paper to be tested.
n Paper
0, 48 Basic sheet (paper roll)
1, 49
2, 50 Paper roll
m specifies a test pattern.
m Test pattern
1, 49 Hexadecimal dump
2, 50 Printer status print
3, 51 Rolling pattern print

[Description]
· This command is enabled only when processed at the beginning of a line in standard
mode.
· This command is no effect in page mode.
· When this command is received during macro definition, the printer ends macro definition
and begins performing this command.
· After the test print is finished, the printer resets itself automatically. Therefore, the
already-defined data before this command is executed, such as an user-defined
characters, downloaded bit image, and macro, becomes undefined,
and the receive buffer and print buffer are cleared, and each setting returns to the default
value. The printer also re-reads the DIP switch settings.
· The printer cuts the paper at the end of the test print.
· The printer goes BUSY while this command is executed.

### GS * X Y D1D(X × Y × 8)

[Name]
Define downloaded bit image
[Format]
ASCII GS * x y d1...d(x × y × 8)
Hex 1D 2A x y d1...d(x × y × 8)
Decimal 29 42 x y d1...d(x × y × 8)
[Range]
1 ≤ x ≤ 255, 1 ≤ y ≤ 255
x × y ≤ 1024
0 ≤ d ≤ 255
[Description]
Defines a downloaded bit image using the number of bytes specified by x and y
· x specifies the number of dots in the horizontal direction.
· y specifies the number of dots in the vertical direction.
[Notes]
· The number of dots in the horizontal direction is x × 8, in the vertical direction itis y × 8.
· If x × y is out of the specified range, this command is disabled.
· The d indicates bit-image data. Data ( d) specifies a bit printed to 1 and not printed to 0.
· The downloaded bit image definition is cleared when:


```
① Printer is reset or the power is turned off.
· The following figure shows the relationship between the downloaded bit image and the
printed data.
```
[Reference]
GS /

### GS / M

[Name]
Print downloaded bit image
[Format]
ASCII GS / m
Hex 1D 2F m
Decimal 29 47 m
[Range]
0 ≤ m ≤ 3, 48 ≤ m ≤ 51
[Description]
Prints a downloaded bit image using the mode specified by m.
m selects a mode from the table below:
m Mode Vertical Dot Density (DPI) Horizontal Dot Density (DPI)
0, 48 Normal 180 203
1, 49 Double-width 180 101
2, 50 Double-height 90 203
3, 51 Quadruple 90 101

[Notes]
· This command is ignored if a downloaded bit image has not been defined.
· In standard mode, this command is effective only when there is no data in the print buffer.
· This command has no effect in the print modes (emphasized, double-strike, underline,
character size, or white/black reverse printing), except for upside-down printing mode.
· If the downloaded bit-image to be printed exceeds the printable area, the excess data is
not printed.
[Reference]
GS *

### GS :

[Name]
Start/end macro definition
[Format]
ASCII GS :
Hex 1D 3A
Decimal 29 58
[Description]
Starts or ends macro definition.


[Notes]
·Macro definition starts when this command is received during normal operation. Macro
definition ends when this command is received during macro definition.
· When GS ^ is received during macro definition, the printer ends macro definition and
clears the definition.
· Macro is not defined when the power is turned on.
· The defined contents of the macro are not cleared by ESC @. Therefore, ESC @ can be
included in the contents of the macro definition.
· If the printer receives GS : again immediately after previously receiving GS :, the printer
remains in the macro undefined state.
· The contents of the macro can be defined up to 2048 bytes. If the macro definition
exceeds 2048 bytes, excess data is not stored.
[Reference]
GS ^

### GS B N

[Name]
Turn on or off white/black reverse printing mode
[Format]
ASCII GS B n
Hex 1D 42 n
Decimal 29 66 n
[Range]
0 ≤ n ≤ 255
[Description]
Turns on or off white/black reverse printing mode.
· When the LSB of n is 0, white/black reverse mode is turned off.
· When the LSB of n is 1, white/black reverse mode is turned on.
[Notes]
· Only the lowest bit of n is valid.
· This command is available for built-in characters and user-defined characters.
· When white/black reverse printing mode is on, it also applied to character spacing set by
ESC SP.
· This command does not affect bit image, user-defined bit image, bar code, HRI
characters, and spacing skipped by HT, ESC $, and ESC \.
· This command does not affect the space between lines.
· White/black reverse mode has a higher priority than underline mode. Even if underline
mode is on, it is disabled (but not canceled) when white/black reverse mode is selected.
[Default] n = 0

### GS H N

[Name]
Select printing position for HRI characters
[Format]
ASCII GS H n
Hex 1D 48 n
Decimal 29 72 n
[Range]
0 ≤ n ≤ 3, 48 ≤ n ≤ 51
[Description]
Selects the printing position of HRI characters when printing a bar code.


n selects the printing position as follows:
n Printing position
0, 48 Not printed
1, 49 Above the bar code
2, 50 Below the bar code
3, 51 Both above and below the bar code
· HRI indicates Human Readable Interpretation.
[Notes]
· HRI characters are printed using the font specified by GS f.
[Default] n = 0
[Reference]
GS f, GS k

### GS I N

[Name]
Transmit printer ID
[Format]
ASCII GS I n
Hex 1D 49 n
Decimal 29 73 n
[Range]
1 ≤ n ≤ 3, 49 ≤ n ≤ 51, 65 ≤ n ≤ 69
[Description]
Transmits the printer ID specified by n as follows:
n Printer ID Specification ID (hexadecimal)
1,49 Printer model ID BTP2002 series 20
2,50 Type ID See table below.
3,51 ROM version ID Depends on ROM version.
65 Firmware version Depends on Firmware version.
66 Manufacturer SNBC
67 Printer name BTP-2002NP
68 Serial number Depends on serial number.
69 Supporting Kanji type Japan model: KANJI JAPANESE
China model: CHINA GB2312
Taiwan model: TAIWAN BIG-5
Thai model: THAI 3 PASS

n = 2, Type ID
Bit Off/On Hex Decimal Function
0 OFF^00 0 Two-byte character code not supported.
ON 01 1 Two-byte character code supported.
1 ON 02 2 Auto-cutter equipped.
2 OFF 00 0 No direct connection with customer display
3 OFF 00 0 No MICR reader.
4 OFF 00 0 Not used. Fixed to Off.
5 _ _ _ Undefined.
6 _ _ _ Undefined.
7 OFF 00 0 Not used. Fixed to Off.


[Notes]
The printer ID is transmitted when the data in the receive buffer is developed. Therefore, there
may be a time lag between receiving this command and transmitting the status, depending on
the receive buffer status.
· When the printer ID transmission is specified with (1 ≤ n ≤ 3) or (49 ≤ n ≤ 51), one byte code is
transmitted.
· When Auto Status Back (ASB) is enabled using GS a, the status transmitted by GS I and the
ASB status must be differentiated.
· When the printer ID transmission is specified with (65 ≤ n ≤ 69), the following contents are
transmitted:
Header: Hexadecimal = 5FH / Decimal = 95 (1 byte)
Data: Printer information
NUL: Hexadecimal = 00H / Decimal = 0 (1 byte)

### GS L NL NH

[Name]
Set left margin
[Format]
ASCII GS L nL nH
Hex 1D 4C nL nH
Decimal 29 76 nL nH
[Range]
0 ≤ nL ≤ 255
0 ≤ nH ≤ 255
[Description]
Sets the left margin using nL and nH.
· The left margin is set to [( nL + nH × 256) × horizontal motion unit)] inches.

[Notes]
· This command is effective only processed at the beginning of the line in standard mode.
· If this command is input in page mode, the printer performs only internal flag operations.
· This command does not affect printing in page mode.
· If the setting exceeds the printable area, the maximum value of the printable area is used.
· The horizontal and vertical motion units are specified by GS P. Changing the horizontal
and vertical motion unit does not affect the current left margin.
[Default] nL = 0, nH = 0
[Reference]
GS P, GS W

### GS P X Y

[Name]
Set horizontal and vertical motion units
[Format]
ASCII GS P x y
Hex 1D 50 x y
Decimal 29 80 x y
[Range]
0 ≤ x ≤ 255
0 ≤ y ≤ 255


[Description]
Sets the horizontal and vertical motion units to approximately 25.4/ x mm { 1/ x inches}
and approximately 25.4/ y mm {1/ y inches}, respectively.
When x and y are set to 0, the default setting of each value is used.
[Notes]
The horizontal direction is perpendicular to the paper feed direction and the vertical
direction is the paper feed direction.
· In standard mode, the following commands use x or y, regardless of character rotation
(upside-down or 90° clockwise rotation):
①Commands using x: ESC SP, ESC $, ESC \, FS S, GS L, GS W
②Commands using y: ESC 3, ESC J, GS V
· In page mode, the following command use x or y, depending on character orientation:
① When the print starting position is set to the upper left or lower right of the printing area
using ESC T (data is buffered in the direction perpendicular to the paper feed direction):
Commands using x: ESC SP, ESC $, ESC W, ESC \, FS S
Commands using y: ESC 3, ESC J, ESC W, GS $, GS \, GS V
② When the print starting position is set to the upper right or lower left of the printing area
using ESC T (data is buffered in the paper feed direction):
Commands using x: ESC 3, ESC J, ESC W, GS $, GS \
Commands using y: ESC SP, ESC $, ESC W, ESC \,FS S, GS V
· The command does not affect the previously specified values.
[Default] x = 203, y = 180
[Reference]
ESC SP, ESC $, ESC 3, ESC J, ESC W, ESC \, GS $, GS L, GS V, GS W, GS \

①②GS V m GS V m n

[Name]
Select cut mode and cut paper
[Format]
ASCII ① GS V m
Hex 1D 56 m
Decimal 29 86 m
②.ASCII GS V m n
Hex 1D 56 m n
Decimal 29 86 m n
[Range]
①m = 1, 49
②m = 66, 0 ≤ n ≤ 255
[Description]
Selects a mode for cutting paper and executes paper cutting. The value of m
selects the mode as follows:
m Print mode
0,48 Full cut
1,49 Partial cut (one point left uncut)
66 Feeds paper (cutting position + [ n × (vertical motion unit)]), and cuts the paper partially (one point left uncut).

[Notes for and ]①②
· This command is effective only processed at the beginning of a line.


[Note for ]② · When n = 0, the printer feeds the paper to the cutting position and cuts it.
· Normally, the printer feeds the paper to (cutting position + [ n × vertical motion unit]) and
cuts it.
· The horizontal and vertical motion unit are specified by GS P.

### GS W NL NH

[Name]
Set printing area width
[Format]
ASCII GS W nL nH
Hex 1D 57 nL nH
Decimal 29 87 nL nH
[Range]
0 ≤ nL ≤ 255
0 ≤ nH ≤ 255
[Description]
Sets the printing area width to the area specified by nL and nH.
· The printing area width is set to [( nL + nH × 256) × horizontal motion unit)] inches.

[Notes]
· This command is effective only processed at the beginning of the line.
· In page mode, the printer performs only internal flag operations.
· This command does not affect printing in page mode.
· If the [left margin + printing area width] exceeds the printable area, [printable area width -
left margin) is used.
· The horizontal and vertical motion units are specified by GS P. Changing the horizontal
and vertical motion units does not affect the current left margin.
· The horizontal motion unit ( x) is used for calculating the printing area width. The
calculated result is truncated to the minimum value of the mechanical pitch.
[Default] nL = 80, nH = 2
[Reference]
GS L, GS P

### GS \ NL NH

[Name]
Set relative vertical print position in page mode
[Format]
ASCII GS \ nL nH
Hex 1D 5C nL nH
Decimal 29 92 nL nH
[Range]
0 ≤ nL ≤ 255
0 ≤ nH ≤ 255
[Description]
Sets the relative vertical print starting position from the current position in page mode.
· This command sets the distance from the current position to [( nL + nH × 256) × vertical or
horizontal motion unit] inches.
[Notes]
· This command is ignored unless page mode is selected.


· When pitch N is specified to the movement downward:
nL + nH × 256 = N
When pitch N is specified to the movement upward (the negative direction), use the
complement of 65536.
When pitch N is specified to the movement upward:
nL + nH × 256 = 65536 - N
· Any setting that exceeds the specified printing area is ignored.
· This command function as follows, depending on the print starting position set by ESC T:
① When the starting position is set to the upper left or lower right of the printing, the
vertical motion unit (y) is used.
② When the starting position is set to the upper right or lower left of the printing area, the
horizontal motion unit (x) is used.
· The horizontal and vertical motion unit are specified by GS P.
[Reference]
ESC $, ESC T, ESC W, ESC \, GS $, GS P, 3.12 Page Mode

### GS ^ R T M

[Name]
Execute macro
[Format]
ASCII GS ^ r t m
Hex 1D 5E r t m
Decimal 29 94 r t m
[Range]
0 ≤ r ≤ 255
0 ≤ t ≤ 255
m = 0, 1
[Description]
Executes a macro.
· r specifies the number of times to execute the macro.
· t specifies the waiting time for executing the macro.
· m specifies macro executing mode.
When the LSB of m = 0:
The macro executes r times continuously at the interval specified by t.
When the LSB of m = 1:
After waiting for the period specified by t, the PAPER OUT LED indicators blink ,and the
printer waits for the FEED button to be pressed. After the button is pressed, the printer
executes the macro once. The printer repeats the operation r times.
[Notes]
· The waiting time is t × 100 ms for every macro execution.
· If this command is received while a macro is being defined, the macro definition is
aborted and the definition is cleared.
· If the macro is not defined or if r is 0, nothing is executed.
· When the macro is executed ( m = 1), paper always cannot be fed by using the FEED
button.
[Reference]
GS :


### GS A N

[Name]
Enable/Disable Automatic Status Back (ASB)
[Format]
ASCII GS a n
Hex 1D 61 n
Decimal 29 97 n
[Range]
0 ≤ n ≤ 255
[Description]
Enables or disables ASB and specifies the status items to include, using n as follows:
Bit Off/On Hex Decimal Status for ASB
0 off^00 0 Drawer kick-out connector pin 3 status disabled.
on 01 1 Drawer kick-out connector pin 3 status enabled.
1 off^00            0 On-line/off-line status disabled.
on 02 2 On-line/off-line status enabled.
2 off^00 0 Error status disabled.
on 04 4 Error status enabled.
3 off^00 0 Paper roll sensor status disabled.
on 08 8 Paper roll sensor status enabled.
4-7 - - - Undefined.

[Notes]
If any of the status items in the table above are enabled, the printer transmits the status
when this command is executed. The printer automatically transmits the status whenever
the enabled status item changes. The disabled status items may change, in this case,
because each status transmission represents the current status.
If all status items are disabled, the ASB function is also disabled.
The following four status bytes are transmitted without confirming whether the host is ready
to receive data. The four status bytes must be consecutive, except for the XOFF code.
Since this command is executed after the data is processed in the receive buffer, there
may be a time lag between data reception and status transmission.
When the printer is disabled by ESC = (Select peripheral device), the four status bytes are
transmitted whenever the status changes.
When using DLE EOT, GS I, or GS r, the status transmitted by these commands and ASB
status must be differentiated.
The status to be transmitted are as follows:


```
First byte (printer information)
Bit Off/On Hex Decimal Status for ASB
0 Off 00 0 Not used. Fixed to Off.
1 Off 00 0 Not used. Fixed to Off.
2 Off^000 Drawer kick-out connector pin 3 is LOW.^
On 04 4 Drawer kick-out connector pin 3 is HIGH.
3 Off^000 On-line.^
On 08 8 Off-line.
4 On 10 16 Not used. Fixed to On.
5 Off^000 Cover is closed.^
On 20 32 Cover is open.
6 Off^000 Paper is not being fed by using the PAPER FEED button.^
On 40 64 Paper is being fed by using the PAPER FEED button.
7 Off 00 0 Not used. Fixed to Off.
Second byte (printer information)
Bit Off/On Hex Decimal Status for ASB
0 - - - Undefined.
1 - - - Undefined.
2 - - - Undefined.
3 Off^000 No auto cutter error.^
On 08 8 Auto cutter error occurred.
4 Off 00 0 Not used. Fixed to Off.
5 Off^000 No unrecoverable error.^
On 20 32 Unrecoverable error occurred.
6 Off^000 No automatically recoverable error.^
On 40 64 Automatically recoverable error occurred.
7 Off 00 0 Not used. Fixed to Off.
```
Bit 5: If these errors occur due to paper jams or the like, it is possible to recover by correcting the
cause of the error and executing DLE ENQ n (1 ≤ n ≤ 2). If an error due to a circuit failure (e.g.
wire break) occurs, it is impossible to recover.
Bit 6: When printing is stopped due to high print head temperature until the print head temperature
drops sufficiently or when the paper roll cover is open during printing, bit 6 is On.
Third byte (paper sensor information)
Bit Off/On Hex Decimal Status for ASB
0,1 Off^00 0 Paper roll near-end sensor: paper adequate.^
On 03 3 Paper roll near-end sensor: paper near end.
2,3 Off^00 0 Paper roll end sensor: paper present^
On 0C 12 Paper roll end sensor: paper not present
4 Off 00 0 Not used. Fixed to Off.
5,6 - - - Undefined.
7 Off 00 0 Not used. Fixed to Off.


```
Fourth byte (paper sensor information)
Bit Off/On Hex Decimal Status for ASB
0-3 - - - Undefined.
4 Off 00 0 Not used. Fixed to Off.
5,6 - - - Undefined.
7 Off 00 0 Not used. Fixed to Off.
```
### GS F N

[Name]
Select font for Human Readable Interpretation (HRI) characters
[Format]
ASCII GS f n
Hex 1D 66 n
Decimal 29 102 n
[Range]
n = 0, 1, 48, 49
[Description]
Selects a font for the HRI characters used when printing a bar code.
n selects a font from the following table:
n Font
0,48 Font A (12 × 24)
1,49 Font B (9 × 17)

[Notes]
· HRI indicates Human Readable Interpretation.
· HRI characters are printed at the position specified by GS H.
[Default] n = 0
[Reference]
GS H, GS k

### GS H N

[Name]
Select bar code height
[Format]
ASCII GS h n
Hex 1D 68 n
Decimal 29 104 n
[Range]
1 ≤ n ≤ 255
[Description]
Selects the height of the bar code.
n specifies the number of dots in the vertical direction.
[Default] n = 162
[Reference]
GS k

①②GS k m d1...dk NUL GS k m n d1...dn

[Name]
Print bar code
[Format]
ASCII ① GS k m d1...d k NUL
Hex 1D 6B m d1...d k 00
Decimal 29 107 m d1...d k 0


②ASCII GS k m n d1... dn
Hex 1D 6B m n d1... dn
Decimal 29 107 m n d1... dn
[Range]
0 ① ≤ m ≤ 6, m = 10 (k and d depends on the bar code system used)
② 65 ≤ m ≤ 73 , m = 75 (n and d depends on the bar code system used)
[Description]
Selects a bar code system and prints the bar code.
m selects a bar code system as follows:

m Bar Code System (^) CharactersNumber of Remarks
0 UPC-A 11 ≤ k ≤ 12 48 ≤ d ≤ 57
1 UPC-E 11 ≤ k ≤ 12 48 ≤ d ≤ 57
2 JAN13 (EAN13) 12 ≤ k ≤ 13 48 ≤ d ≤ 57
3 JAN 8 (EAN8) 7 ≤ k ≤ 8 48 ≤ d ≤ 57
4 CODE39 1 ≤ k ≤ 255 45 ≤ d ≤ 57, 65 ≤ d ≤ 90, 32, 36, 37,43
5 ITF 1 ≤ k ≤ 255 48 ≤ d ≤ 57
6 CODABAR 1 ≤ k ≤ 255 48 ≤ d ≤ 57, 65 ≤ d ≤ 68 , 36, 43, 45,46,47,58
①
10 PDF 417 1 ≤ k ≤ 255 32 ≤ d ≤ 255
65 UPC-A 11 ≤ n ≤ 12 48 ≤ d ≤ 57
66 UPC-E 11 ≤ n ≤ 12 48 ≤ d ≤ 57
67 JAN13 (EAN13) 12 ≤n ≤ 13 48 ≤ d ≤ 57
68 JAN 8 (EAN8) 7 ≤n ≤ 8 48 ≤ d ≤ 57
69 CODE39 1 ≤ n ≤ 255 45 ≤ d ≤ 57, 65 ≤ d ≤ 90, 32, 36, 37,43
d1 = dk = 42 (1)
70 ITF 1 ≤ n ≤ 255 48 ≤ d ≤ 57
71 CODABAR 1 ≤ n ≤ 255 48 ≤ d ≤ 57 65 ≤ d ≤ 68, 36, 43,45,46,47 58
72 CODE93 1 ≤ n ≤ 255 0 ≤ d ≤ 127
73 CODE128 2 ≤ n ≤ 255 0 ≤ d ≤ 127
②
75 PDF417 1 ≤ n ≤ 255 0 ≤ d ≤ 255
[Notes for ]①
· This command ends with a NUL code.
· The number of data for ITF bar code must be even numbers. When an odd number of
data is input, the printer ignores the last received data.
[Notes for ]②
· n indicates the number of bar code data, and the printer processes n bytes from
the next character data as bar code data.
· If n is outside of the specified range, the printer stops command processing and
processes the following data as normal data.
[Notes in standard mode]
· If d is outside of the specified range, the printer only feeds paper and processes
the following data as normal data.
· If the horizontal size exceeds printing area, the command is ignored.
· This command feeds as much paper as is required to print the bar code,


regardless of the line spacing specified by ESC 2 or ESC 3.
· This command is enabled only when no data exists in the print buffer. When
data exists in the print buffer, the printer processes the data following m as
normal data.
· After printing bar code, this command sets the print position to the beginning of
the line.
· This command is not affected by print modes (emphasized, double-strike,
underline, character size, white/black reverse printing, or 90° rotated character,
etc.), except for upside-down printing mode.
[Notes in page mode]
· This command develops bar code data in the print buffer, but does not print it. After
processing bar code data, this command moves the print position to the right side dot of
the bar code.
· If d is out of the specified range, the printer stops command processing and processes
the following data as normal data. In this case the data buffer position does not change.
· If bar code width exceeds the printing area, the printer does not print the bar code
When CODE128 ( m = 73) is used:
· Refer to Appendix A for the information of the CODE 128 bar code and its code table.
· When using the CODE 128 in this printer, take the following points into account for data
transmission:
① The top of the bar code data string must be code set selection character (any of
CODE A, CODE B or CODE C) which selects
② Special characters are defined by combining two characters "{" and one character.
The ASCII character "{" is defined by transmitting "{" twice consecutively.
Specific character Transmit data^
ASCII Hex Decimal
SHIFT {S 7B, 53 123,83
CODE A {A 7B, 41 123, 65
CODE B {B 7B, 42 123, 66
CODE C {C 7B, 43 123, 67
FNC1 {1 7B, 31 123, 49
FNC2 {2 7B, 32 123, 50
FNC3 {3 7B, 33 123, 51
FNC4 {4 7B, 34 123, 52
"{" {{ 7B, 7B 123, 123

[Example] Example data for printing "No. 123456"
In this example, the printer first prints "No." using CODE B, then prints the
following numbers using CODE C.
GS k 73 10 123 66 78 111 46 123 67 12 34 56


· If the top of the bar code data is not the code set selection character, the printer stops
command processing and ignore the following data.
· If combination of "{" and the following character does not apply any special character, the
printer stops command processing and ignore the following data.
· If the printer receives characters that cannot be used in the special code set, the printer
stops command processing and ignore the following data.
· The printer does not print HRI characters that correspond to the shift characters or code
set selection characters.
· HRI character for the function character is space.
· HRI characters for the control character (<00>H to <1F>H and <7F>H) are not printed.
<Others> Be sure to keep spaces on both right and left sides of a bar code. (Spaces are different
depending on the types of the bar code.)
[Reference]
GS H, GS f, GS h, GS w.

### GS P N

[Name]
Set all the parameters to define PDF417
[Format]
_ASCII GS p nA nB nC nD nE nF
Hex 1D 70 nA nB nC nD nE nF
Decimal 29 112 nA nB nC nD nE nF_
[Range]
_1 ≤ nA ≤ 10_ ， _1 ≤nB≤ 100_ ， _3 ≤nC ≤ 90_ ， _1 ≤nD ≤ 30_ ， _1 ≤nE ≤ 7_ ， _2 ≤nF ≤ 25_
[Description]
The meaning of parameter n is shown as below:
Parameter Meaning
_nA_ Aspect scale factor of height
_nB_ Aspect Scale factor of width
_nC_ Number of rows
_nD_ Number of columns
_nE_ Width of basic cells
_nF_ Height of basic cells

[Notes]
nA and nB is effective when nC and nD equals to zero. The printer will automatically adjust
the numbers of rows and columns when nA and nB is effective.

### GS Q N

[Name]
Set error correction grade of PDF417 code
[Format]
ASCII GS q _n_
Hex 1D 71 _n_
Decimal 29 113 _n_
[Range]
0 ≤ _n_ ≤ 8
[Description]
Set error correction grade of PDF417 code.


### GS R N

[Name]
Transmit status
[Format]
ASCII GS r n
Hex 1D 72 n
Decimal 29 114 n
[Range]
n = 1, 2, 49, 50
[Description]
Transmits the status specified by n as follows:

(^) n Function
1, 49 Transmits paper sensor status
2, 50 Transmits drawer kick-out connector status
[Notes]
This command is valid for serial model only, The printer transmits only 1 byte without
confirming the condition of the DSR signal.
· This command is executed when the data in the receive buffer is developed. Therefore,
there may be a time lag between receiving this command and transmitting the status,
depending on the receive buffer status.
· The status types to be transmitted are shown below:
Paper sensor status ( n = 1, 49):
Bit Off/On Hex Decimal Status for ASB
0, 1 Off^00 0 Paper roll near-end sensor: paper adequate
On 03 3 Paper roll near-end sensor: paper near end
2, 3 Off^00 0 Paper roll end sensor: paper adequate
On 0c 12 Paper roll end sensor: paper near end
4 Off 00 0 Not used. Fixed to Off
5,6 Undefined
7 Off 00 0 Not used. Fixed to Off
Bits 2 and 3: When the paper end sensor detects a paper end, the printer goes off-line and
does not execute this command. Therefore, bits 2 and 3 do not transmit the status of paper
end.
Drawer kick-out connector status ( n = 2, 50):
Bit Off/On Hex Decimal Status for ASB
0 Off 00 0 Drawer kick-out connector pin 3 is LOW
(^) On 01 1 Drawer kick-out connector pin 3 is HIGH
1- 3 Undefined
4 Off 00 0 Not used. Fixed to Off
5,6 Undefined
7 Off 00 0 Not used. Fixed to Off.
[Reference]
DLE EOT, GS a


### GS V 0 M XL XH YL YH D1DK

[Name]
Print raster bit image
[Format]
ASCII GS v 0 m xL xH yL yH d1...dk
Hex 1D 76 30 m xL xH yL yH d1...dk
Decimal 29 118 48 m xL xH yL yH d1...dk
[Range]
0 ≤ m ≤ 3, 48 ≤ m ≤ 51
0 ≤ xL ≤ 255
0 ≤ xH ≤ 255
0 ≤ yL ≤ 255
0 ≤ d ≤ 255
k = ( xL + xH × 256) × ( yL + yH × 256) ( k ≠ 0)
[Description]
Selects Raster bit-image mode. The value of m selects the mode, as follows:
m Mode Vertical Dot Density (DPI) Horizontal Dot Density (DPI)
0, 48 Normal 180 DPI 203 DPI
1, 49 Double-width 180 DPI 101 DPI
2, 50 Double-height 90 DPI 203 DPI
3, 51 Quadruple 90 DPI 101 DPI
· xL, xH, select the number of data bytes ( xL+ xH × 256) in the horizontal direction for the
bit image.
· yL, yH, select the number of data bytes ( yL+ yH × 256) in the vertical direction for the bit
image.
[Notes]
· In standard mode, this command is effective only when there is no data in the print buffer.
· This command has no effect in all print modes (character size, emphasized, double-strike,
upside-down, underline, white/black reverse printing, etc.) for raster bit image.
· Data outside the printing area is read in and discarded on a dot-by-dot basis.
· The position at which subsequent characters are to be printed for raster bit image is
specified by HT (Horizontal Tab), ESC $ (Set absolute print position),
ESC \ ( Set relative print position), and GS L (Set left margin ). If the position at which
subsequent characters are to be printed is not a multiple of 8, print speed may decline.
· The ESC a (Select justification) setting is also effective on raster bit images.
· When this command is received during macro definition, the printer ends macro definition,
and begins performing this command. The definition of this command should be cleared.
· d indicates the bit-image data. Set a bit to 1 prints a dot and setting it to 0 does not print a
dot.
[Example] When xL+ xH × 256 = 64


### GS W N

[Name]
Set bar code width
[Format]
ASCII GS w n
Hex 1D 77 n
Decimal 29 119 n
[Range]
2 ≤ n ≤ 6
[Description]
Set the horizontal size of the bar code.
n specifies the bar code width as follows:
n Module Width (mm) for Binary-level Bar Code^
Multi-level Bar Code Thin element width (mm) Thick element width (mm)
2 0.25 0.25 0.625
3 0.375 0.375 1.0
4 0.5 0.5 1.25
5 0.625 0.625 1.625
6 0.75 0.75 1.875
· Multi-level bar codes are as follows:
UPC-A, UPC-E, JAN13 (EAN13), JAN8 (EAN8), CODE93, CODE128
· Binary-level bar codes are as follows:
CODE39, ITF, CODABAR
[Default] n = 2
[Reference]
GS k

### GS { W

[Name]
Enable\Disable watermark mode
[Format]
ASCII GS { w n
Hex 1D 7B 77 n
Decimal 29 123 119 n
[Range ] n = 0、 1 。
[Description]n = 0：Enable watermark mode;
n = 1：Disable watermark mode.
[Note]
• This command should be used at the beginning of the line, otherwise it is not
effective.

- Please use GS { w f to define the bitmap before using this command.
- When disable watermark mode using this command, the printer recovers to the
normal print mode.

### GS { W F

[Name]
Set watermark bitmap parameters and enter watermark mode.
[Format]
ASCII GS { w f n1 n2 n3 n4 n5
Hex 1D 7B 77 02 n1 n2 n3 n4 n5
Decimal 29 123 119 02 n1 n2 n3 n4 n5
[Range]
n1 = 0、 1 ；
n2 = 0、 1 、 2 ；


```
n3, using the value in figure 1
[Description]n1 specifies watermark printing mode:
0 ：To print watermark when paper feed
1: To print watermark when print start
n2 specifies watermark aligning mode:
0: Align to left side
1: Centralized
2: Align to right side
```
- n3 specifies Watermark enlargement option shown as figure 1:
    Bit 0/1 Hex Decimal Function
0-3 Character height option (Refer to Figure2)
4-7 Character width option (Refer to Figure2)
Figure 1

```
Character height option Character width option
Hex Decimal Horizontal Hex Decimal Vertical
10 16 1 （Normal） 01 1 1 （Normal）
20 32 2 （Double width） 02 2 2 （Double height）
30 48 3 03 3 3
40 64 4 04 4 4
50 80 5 05 5 5
60 96 6 06 6 6
Figure 2
```
- n4 specifies Watermark brightness and recommended value is 0x20.
- n5 specifies number of bitmap in Flash (Defined by FS q).

[Note]
• This command should be used at the beginning of the line, otherwise it is not effective.

- This command is only effective in line mode and not effective in page mode.
- Please use FS q to define the bitmap before using this command.

[Example] 1D 7B 77 02 01 00 22 40 01
Explanation:
n1=0x01; Print watermark when print start
n2=0x00; Align to left side
n3=0x22 ; Double width and double height
n4=0x40; Watermark brightness is 0x40
n1=0x01; Use the number 1 bitmap in Flash as watermark image

## 3 Kanji Control Commands .................................................................................................................

### FS! N

[Name]
Set print mode(s) for Kanji characters
[Format]
ASCII FS! n


Hex 1C 21 n
Decimal 28 33 n
[Range]
0 ≤n ≤ 255
[Description]
Sets the print mode for Kanji characters, using n as follows:
Bit Off/On Hex Decimal Status for ASB
0, 1 Undefined
2 Off^00 0 Double-width mode is OFF
On 04 4 Double-width mode is ON
3 Off^00 0 Double-height mode is OFF.
On 08 8 Double-height mode is ON
4-6 Undefined
7 Off^00 0 Underline mode is OFF
On 80 128 Underline mode is ON

[Notes]
When both double-width and double-height modes are set (including right- and left-side
character spacing), quadruple-size characters are printed.
· The printer can underline all characters (including right- and left-side character spacing),
but cannot underline the space set by HT and 90° clockwise-rotated characters.
· The thickness of the underline is that specified by FS -, regardless of the character size.
· When some of the characters in a line are double or more height, all the characters on the
line are aligned at the baseline.
· It is possible to emphasize the Kanji character using FS W or GS !, the setting of the last
received command is effective.
· It is possible to turn under line mode on or off using FS -, and the setting of the last
received command is effective.
[Default] n = 0
[Reference]
FS-, FS W, GS!

### FS &

[Name]
Select Kanji character mode
[Format]
ASCII FS &
Hex 1C 26
Decimal 28 38
[Description]
Selects Kanji character mode.
[Notes]
When the kanji character mode is selected, the printer checks whether the code is for
Kanji or not, then processed the first byte and the second byte if the code is for Kanji.
· Kanji codes are processed in the order of the first byte and second byte.
· Kanji character mode is not selected when the power is turned on.
[Reference]
FS ., FS C

### FS - N

[Name]
Turn underline mode on/off for Kanji characters


[Format]
ASCII FS - n
Hex 1C 2D n
Decimal 28 45 n
[Range]
0 ≤ n ≤ 2, 48 ≤ n ≤ 50
[Description]
Turns underline mode for Kanji characters on or off, based on the following values
of n.
n Function
0, 48 Turns off underline mode for Kanji characters
1, 49 Turns on underline mode for Kanji characters (1-dot thick)
2, 50 Turns on underline mode for Kanji characters (2-dot thick)

[Notes]
The printer can underline all characters (including right- and left-side character spacing),
but cannot underline the space set by HT and 90° clockwise-rotated characters.
After the underline mode for Kanji characters is turned off by setting n to 0, underline
printing is no longer performed, but the previously specified underline thickness is not
changed. The default underline thickness is 1 dot.
The specified line thickness does not change even when the character size changes.
It is possible to turn underline mode on or off using FS !, and the last received command is
effective.
[Default] n = 0
[Reference]
FS!

### FS

[Name]
Cancel Kanji character mode
[Format]
ASCII FS.
Hex 1C 2E
Decimal 28 46
[Description]
Cancels Kanji character mode.
[Notes]
For Chinese Kanji model:
When the Kanji character mode is not selected, all character codes are processed one
byte at a time as ASCII code.
Kanji character mode is selected when the power is turned on.
[Reference]
FS &, FS C


### FS 2 C 1 C 2 D1DK

[Name]
Define user-defined Kanji characters
[Format]
ASCII FS 2 c1 c2 d1...dk
Hex 1C 32 c1 c2 d1...dk
Decimal 28 50 c1 c2 d1...dk
[Range]
c1 and c2 indicate character codes for the defined characters.
c1 = FEH, A1H ≤ c2 ≤ FEH
0 ≤ d ≤ 255
k = 72
[Description]
Defines user-defined Kanji characters for the character codes specified by c1 and c2.
[Notes]
c1 and c2 indicate character codes for the defined characters. c1 specifies for the first byte,
and c2 for the second byte.
d indicates the dot data. Set a corresponding bit to 1 to print a dot or to 0 to not print a dot.
[Default] All spaces.
[Reference]
FS C


### FS C N

[Name]
Select Kanji character code system
[Format]
ASCII FS C n
Hex 1C 43 n
Decimal 28 67 n
[Range]
N = 0,1,48,49
[Description]
Selects a Kanji character code system, based on the following values of n:

```
n Kanji System
0,48 JIS code
1,49 SHIFT JIS code
```
[Notes]
This command is effective only for Japanese model.
In the JIS code system, the following codes are available:
Primary byte: <21>H to <7E>H
Secondary byte: <21>H to <7E>H
 In the SHIFT JIS code system, the following codes are available:
Primary byte: <81>H to <9F>H and <E0>H to <EF>H
Secondary byte: <40>H to <7E>H and <80>H to <FC>H
[Default] n = 0

### FS S N 1 N2

[Name]
Set left- and right-side Kanji character spacing
[Format]
ASCII FS S n1 n2
Hex 1C 53 n1 n2
Decimal 28 83 n1 n2
[Range]
0 ≤ n1 ≤ 255
0 ≤ n2 ≤ 255
[Description]
Sets left-side and right-side Kanji character spacing n1 and n2, respectively.
When the printer model used supports GS P, the left-side character spacing is [ n1 ×
horizontal or vertical motion units], and the right-side character spacing is [ n2 × horizontal
or vertical motion units].
[Notes]
When double-width mode is set, the left- and right-side character spacing is twice the
normal value.
The horizontal and vertical motion units are set by GS P. The previously specified
character spacing does not change, even if the horizontal or vertical motion unit is changed
using GS P.
· In standard mode, the horizontal motion unit is used.
· In page mode, the horizontal or vertical motion unit differs in page mode,
depending on starting position of the printable area as follows:
① When the starting position is set to the upper left or lower right of the
printable area using ESC T, the horizontal motion unit ( x) is used.


② When the starting position is set to the upper right or lower left of the
printable area using ESC T, the vertical motion unit ( y) is used.
③ The maximum right-side spacing is 288/203 inches for the paper roll and is
approximately 36 mm {288/203 inches}. Any setting exceeding the maximum
is converted to the maximum automatically.
[Default] n1 = 0, n2 = 0
[Reference]
GS P

### FS W N

[Name]
Turn quadruple-size mode on/off for Kanji characters
[Format]
ASCII FS W n
Hex 1C 57 n
Decimal 28 87 n
[Range]
0 ≤ n ≤ 255
[Description]
Turns quadruple-size mode on or off for Kanji characters.
· When the LSB of n is 0, quadruple-size mode for Kanji characters is turned off.
· When the LSB of n is 1, quadruple-size mode for Kanji characters is turned on.
[Notes]
· Only the lowest bit of n is valid.
· In quadruple-size mode, the printer prints the same size characters as when
double-width and double-height modes are both turned on.
· When quadruple-size mode is turned off using this command, the following
characters are printed in normal size.
· When some of the characters on a line are different in height, all the characters
on the line are aligned at the baseline.
· FS! or GS! can also select and cancel quadruple-size mode by selecting
double-height and double-width modes, and the setting of the last received
command is effective.
[Default] n = 0
[Reference]
FS !, GS!


## Appendix A: Code128 Bar Code .........................................................................................................

### A1 DESCRIPTION OF THE CODE128 BAR CODE

```
In CODE128 bar code system, it is possible to represent 128 ASCII characters and 2-digit numerals
using one bar code character that is defined by combining one of the 103 bar code characters and 3
code sets. Each code set is used for representing the following characters:
· Code set A: ASCII characters 00H to 5FH
· Code set B: ASCII characters 20H to 7FH
· Code set C: 2-digit numeral characters using one character (100 numerals from 00 to 99)
The following special characters are also available in CODE128:
· SHIFT characters
In code set A, the character just after SHIFT is processed as a character for code set B. In code set
B, the character just after SHIFT is processed as a character for code set A.
SHIFT characters cannot be used in code set C.
· Code set selection character (CODE A, CODE B, CODE C).
This character switches the following code set to code set A, B, or C.
· Function character (FNC1, FNC2, FNC3, FNC4)
The usage of function characters depends on the application software. In code set C, only FNC1 is
available.
```

### A2 CODE TABLES

Printable characters in code set A

```
Character Transmit Data^ Transmit Data^ Transmit Data^
Hex Decimal
```
```
Character
Hex Decimal
```
```
Character
Hex Decimal
NULL
SOH
STX
ETX
EOT
ENQ
ACK
BEL
BS
HT
LF
VT
FF
CR
SO
SI
DLE
DC1
DC2
DC3
DC4
NAK
SYN
ETB
CAN
EM
SUB
ESC
FS
GS
RS
US
SP
! " # $ % & '
```
```
00
01
02
03
04
05
06
07
08
09
0A
0B
0C
0D
0E
0F
10
11
12
13
14
15
16
17
18
19
1A
1B
1C
1D
1E
1F
20
21
22
23
24
25
26
27
```
```
0 1 2 3 4 5 6 7 8 9
```
```
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
```
```
( ) * + , -. / 0 1 2 3 4 5 6 7 8 9 : ; < = >? @ A B C D E F G H I J K L M N O
28
29
2A
2B
2C
2D
2E
2F
30
31
32
33
34
35
36
37
38
39
3A
3B
3C
3D
3E
3F
40
41
42
43
44
45
46
47
48
49
4A
4B
4C
4D
4E
4F
```
```
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
```
```
P Q R S T U V W X Y Z [ \ ] ^ _
```
```
FNC1
FNC2
FNC3
FNC4
SHIFT
CODEB
CODEC
```
```
50
51
52
53
54
55
56
57
58
59
5A
5B
5C
5D
5E
5F
7B,31
7B,32
7B,33
7B,34
7B,53
7B,42
7B,43
```
```
80
81
82
83
84
85
86
87
88
89
90
91
92
93
94
95
123,49
123,50
123,51
123,52
123,83
123,66
123,67
```

Printable characters in code set B

```
Character Transmit Data^ Transmit Data^ Transmit Data^
Hex Decimal
```
```
Character
Hex Decimal
```
```
Character
Hex Decimal
SP
! " # $ % & ' ( ) * + , -. / 0 1 2 3 4 5 6 7 8 9 : ; < = >? @ A B C D E F G
```
```
20
21
22
23
24
25
26
27
28
29
2A
2B
2C
2D
2E
2F
30
31
32
33
34
35
36
37
38
39
3A
3B
3C
3D
3E
3F
40
41
42
43
44
45
46
47
```
```
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
```
```
H I J K L M N O P Q R S T U V W X Y Z [ \ ] ^ _ ` a b c d e f g h i j k l m n o
48
49
4A
4B
4C
4D
4E
4F
50
51
52
53
54
55
56
57
58
59
5A
5B
5C
5D
5E
5F
60
61
62
63
64
65
66
67
68
69
6A
6B
6C
6D
6E
6F
```
```
72
73
74
75
76
77
78
79
80
81
82
83
84
85
86
87
88
89
90
91
92
93
94
95
96
97
98
99
100
101
102
103
104
105
106
107
108
109
110
111
```
```
p q r s t u v w x y z { | } —
```
```
DEL
FNC1
FNC2
FNC3
FNC4
SHIFT
CODEA
CODEC
```
```
70
71
72
73
74
75
76
77
78
79
7A
7B,7B
7C
7D
7E
7F
7B,31
7B,32
7B,33
7B,34
7B,53
7B,41
7B,43
```
```
112
113
114
115
116
117
118
119
120
121
122
123,123
124
125
126
127
123,49
123,50
123,51
123,52
123,83
123,65
123,67
```

Printable characters in code set C

```
Character Transmit Data^ Transmit Data^ Transmit Data^
Hex Decimal
```
```
Character
Hex Decimal
```
```
Character
Hex Decimal
0 1 2 3 4 5 6 7 8 9
```
```
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
```
```
00
01
02
03
04
05
06
07
08
09
0A
0B
0C
0D
0E
0F
10
11
12
13
14
15
16
17
18
19
1A
1B
1C
1D
1E
1F
20
21
22
23
24
25
26
27
```
```
0 1 2 3 4 5 6 7 8 9
```
```
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
```
```
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
```
```
28
29
2A
2B
2C
2D
2E
2F
30
31
32
33
34
35
36
37
38
39
3A
3B
3C
3D
3E
3F
40
41
42
43
44
45
46
47
48
49
4A
4B
4C
4D
4E
4F
```
```
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
```
```
80
81
82
83
84
85
86
87
88
89
90
91
92
93
94
95
96
97
98
99
FNC1
CODEA
CODEB
```
```
50
51
52
53
54
55
56
57
58
59
5A
5B
5C
5D
5E
5F
60
61
62
63
7B,31
7B,41
7B,42
```
```
80
81
82
83
84
85
86
87
88
89
90
91
92
93
94
95
96
97
98
99
123,49
123,65
123,66
```

## Appendix B: Page Mode ......................................................................................................................

### B1 GENERAL DESCRIPTION

The printer operates in two print modes only when the paper roll is selected as the print sheet: standard
mode and page mode. In standard mode, the printer prints and feeds paper each time it receives print
data or paper feed commands. In page mode, all the received print data and paper feed commands are
processed in the specified memory, and the printer executes no operations. All the data in the memory is
then printed when an ESC FF or FF command is received.
For example, when the printer receives the data "ABCDEF" <LF> in standard mode, it prints "ABCDEF"
and feeds the paper by one line. In page mode, "ABCDEF" is written to the specified printing area in
memory, and the position in memory for the next print data is shifted by one line.
The ESC L command puts the printer into page mode, and all commands received thereafter are
processed in page mode. Executing an ESC FF command prints the received data collectively, and
executing an FF command restores the printer to standard mode after the received data is printed
collectively. Executing an ESC S command restores the printer to standard mode without printing the
received data in page mode; the received data is cleared from memory instead.

```
Figure B.1 Shifting Between Standard Mode and Page Mode
```

### B2 SETTING VALUES IN STANDARD AND PAGE MODES

1) The available commands and parameters are the same for both standard and page modes. However,
these values can be set independently in each mode for the ESC SP, ESC 2,ESC 3, and FS S
commands. For these commands, different settings can be stored for each mode.


### B3 FORMATTING OF PRINT DATA IN THE PRINTABLE AREA

Formatting of print data in the printable area is performed as follows:
1) The printable area is set using ESC W. If all printing and feeding are complete before the printer
receives the ESC W command, the left side (as you face the printer) is taken as the origin (x0, y0) of the
printable area. The printable rectangular area is defined by the length (dx dots) extending from and
including the origin (x0, y0) in the x direction (perpendicular to the paper feed direction), and by the
length (dy dots) in the y direction (paper feed direction). (If the ESC W command is not used, the
printable area remains the default value.)
2) When the printer receives print data after ESC W sets the printable area and ESC T sets the printing
direction, the print data is formatted within the printable area so that point A in Figure B.2 is at the
beginning of the printable area as a default value. (When a character is printed, point A is the baseline.)
Print data containing downloaded bit images or bar codes is formatted so that the bottom point of the left
side of the image data (point B in Figure B.3) is aligned with the baseline. However, any Human
Readable Interpretation (HRI) characters are printed under the baseline. At the points labeled Point B, if
characters (such as double-height characters) higher than normal size characters or downloaded bit
image characters are received, any part of the character higher than the normal-size character is not
printed.
3) If the print data (including the space to the right of a character) exceeds the printable area before the
printer receives a command (e.g., LF or ESC J) that includes line feeding, a line feed is executed
automatically within the printable area. The print position, therefore, moves to the beginning of the next
line. The line feed amount depends on the values set by commands (such as ESC 2 and ESC 3).
4) The default value of the line spacing is set to 1/6 inch and corresponds to 31 dots in the vertical
direction. If print data for the next line contains extended characters that are higher than double-height
characters, bit images taking up two or more lines, or bar codes higher than normal characters, the
amount of line feeding may be insufficient, resulting in overlapping of the characters' higher-order dots
with the previous line. To avoid this, increase the amount of line spacing.
Example
When printing a downloaded bit image of six bytes in the vertical direction, use the following formula:
{number of vertical dots (8×6) - number of dots for feeding at the beginning of the printable
area (24)} × vertical motion unit conversions (180/180) = 24
Therefore, 24 dots are required for feeding.
Use the following commands:
ESC W xL, xH, yL, yH, dxL, dxH, dyL, dyH
ESC T n
ESC 3 24 ¬ Set line spacing to be added.
LF
GS / 1
ESC 2 ¬ Reset the line spacing to 1/6 inch.
NOTE: Vertical and horizontal motion units are 1/180 in the vertical direction and 1/203 in the horizontal


direction; therefore, the position you specify varies depending on the printing direction. Setting the
vertical motion unit to 1/180 using the GS P command does not change the current print position.

```
Figure B.2 Character Data Developing Position
```
```
Figure B.3 Print Data Developing Position
```

Figure B.4 Downloaded Bit Image Developing Position


## Appendix C: Character Code Tables ..................................................................................................

### C1 PAGE 0 (PC437: USA)

NOTE: The character code tables show only character configurations. They do not show the actual print
pattern


### C2 PA G E 1 (KATAKANA)


### C3 PAGE 2 ( PC850: MULTILINGUAL)


### C4 PAGE 3 (PC860: PORTUGUESE)


### C5 PAGE 4 (PC863: CANADIAN-FRENCH)


### C6 PAGE 5 (PC865: NORDIC)


### C7 PAGE 18 ( PC852 LAT I N 2 )


### C8 PAGE 19 (PC858)


### C9 PAGE 16 (WPC1252)


### C10 PAGE 17 (PC866: CYRILLIC #2)


### C11 INTERNATIONAL CHARACTER SET


## Control Sequences...............................................................................................................................

The control sequences of the printer controller are POS compatible.
Code Hex Dec Function
HT 09 09 Horizontal tab
LF 0A 10 Print and line feed
FF 0C^12 Print and return to standard mode (in page mode)
CR 0D 13 Print and carriage return
CAN 18 24 Cancel print data in page mode
DLE EOT 10 04 16 04 Real-time status transmission
DLE ENQ 10 05 16 05 Real-time request to printer
DLE DC4 10 14 16 20 Generate pulse at real-time
ESC FF 1B 0C 27 12 Print data in page mode
ESC SP 1B 20 27 32 Set right-side character spacing
ESC ！ 1B 21 27 33 Select print mode(s)
ESC $ 1B 24 27 36 Set absolute print position
ESC % 1B 25 27 37 Select/cancel user-defined character set
ESC & 1B 26 27 38 Define user-defined characters
ESC * 1B 2A 27 42 Select bit-image mode
ESC -n 1B 2D 27 45 Turn underline mode on/off
ESC 2 1B 32 27 50 Select default line spacing
ESC 3 1B 33 27 51 Set line spacing
ESC = 1B 3D 27 61 n Select peripheral device
ESC ？ 1B 3F 27 63 n Cancel user-defined characters
ESC @ 1B 40 27 64 Initialize printer
ESC D 1B 44 27 68 Set horizontal tab positions
ESC E 1B 45 27 69 Turn emphasized mode on/off
ESC G 1B 47 27 71 Turn double-strike mode on/off
ESC J 1B 4A 27 74 n Print and feed paper
ESC L 1B 4C 27 76 Select page mode
ESC M 1B 4D 27 77 Select character font
ESC R 1B 52 27 82 Select an international character set
ESC S 1B 53 27 83 Select standard mode
ESC T 1B 54 27 84 Select print direction in page mode
ESC V 1B 56 27 86 Turn 90° clockwise rotation mode on/off
ESC W 1B 57 27 87 Set printing area in page mode
ESC \ 1B 5C 27 92 Set relative print position
ESC a 1B 61 27 97 Select justification
ESC c 3 1B 63 33 27 99 51 Select paper sensor(s) to output paper-end signals
ESC c 4 1B 63 34 27 99 52 Select paper sensor(s) to stop printing
ESC c 5 1B 63 35 27 99 53 Enable/disable panel buttons
ESC d 1B 64 27 100 Print and feed n lines
ESC p 1B 70 27 112 General pulse


ESC { 1B 7B 27 123 Select character code table
FS g 1 1C 67 31 28 103 49 Write to NV user memory
FS g 2 1C 67 32 28 103 50 Read from NV user memory
FS p 1C 70 28 112 Print NV bit image
FS q 1C 71 28 113 Define NV bit image
GS! 1D 21 29 33 Select character size
GS $ 1D 24 29 36 Set absolute vertical print position in page mode
GS * 1D 2A 29 42 Define downloaded bit image
GS ( A 1D 28 41 29 40 65 Execute test print
GS / 1D 2F 29 47 Print downloaded bit image
GS : 1D 3A 29 58 Start/end macro definition
GS B 1D 42 29 66 Turn white/black reverse printing mode on/off
GS H 1D 48 29 72 Select printing position of HRI characters
GS I 1D 47 29 73 Transmit printer ID
GS L 1D 4C 29 76 Set left margin
GS P 1D 50 29 80 Set horizontal and vertical motion units
GS V 1D 56 29 86 Select cut mode and cut paper
GS W 1D 57 29 87 Set printing area width
GS \ 1D 5C 29 92 Set relative vertical print position in page mode
GS ^ 1D 5E 29 94 Execute macro
GS a 1D 61 29 97 Enable/disable Automatic Status Back (ASB)
GS f 1D 66 29 102 Select font for HRI characters
GS h 1D 68 29 104 Set bar code height
GS k 1D 6B 29 107 Print bar code
GS r 1D 72 29 114 Transmit status
GS v 0 1D 76 30 19 118 48 Print raster bit image
GS w 1D 77 29 119 Set bar code width
GS { w 1D 7B 77 29 123 119 Enable/Disable Water mark Function
GS { w f 1D 7B 77 02 29 123 119 02 Setting Watermark parameter
FS! 1C 21 28 33 Set print mode(s) for Kanji characters
FS & 1C 26 28 38 Select Kanji character mode
FS - 1C 2D 28 45 Turn underline mode on/off for Kanji characters
FS. 1C 2E 28 46 Cancel Kanji character mode
FS2 1C 32 28 50 Define user-defined Kanji characters
FS C 1C 43 28 67 Select Kanji character code system
FS S 1C 53 28 83 Set Kanji character spacing
FS W 1C 57 28 87 Turn quadruple-size mode on/off for Kanji characters


