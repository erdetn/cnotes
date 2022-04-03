# C Notes

C Notes: this is a simple readme note on how I write C code.

- General rules:
  - Indents are one tab (4 spaces).
  - Each line try to keep at most 80 characters long.
  - Comments can be // or /* but // is most commonly used.
  - File names should be lower_case.c.
  - Code should be **consistent**, **maintainable**, **scalable** and **readable**.
  - Code and naming should be designed with the mindset of OOP.

#
- Include header files. 
Group and separate header files into those that comes from include path and local ones, respectively; and/or group headers by libraries

```c
#include <stdlib.h>
#include <stdbool.h>

#include <gtk/gtk.h>
#include <gtk/window.h>

#include "dsp/sound_card.h"
#include "dsp/fir.h"
#include "dsp/circular_buffer.h>
```
#
- Comments: Both `//` and `/* */` are valid, but try to use `//` as much as possible. Avoid using `/* */`. Whenever you start a comment, add a spacing after the `//` then continue with the sentence.
Whenever you have to 'comment' a code, instead of `//` or `/* */`, use `#ifdef` or `#if` directive - either with `0` or name a define that makes sense for that part of the code.

For example, if `#ifdef 0`:
```c
void LcdInitBacklight() {

	SystemCoreClockUpdate();

	LcdInitPwmPinForBacklight(50);

#if 0
	// Keep high the GPIO/PWM for backlight control.
	LcdInitEnablePin(kHighState);
#endif
}
```
or
```c
void LcdInitBacklight() {

	SystemCoreClockUpdate();

	LcdInitPwmPinForBacklight(50);

#if CONFIG_FULL_BACKLIGHT_ENABLED
	// Keep high the GPIO/PWM for backlight control.
	LcdInitEnablePin(kHighState);
#endif            
}
```
#
- Keep macro definitions, enums, global or local variables declarations, function arguments, etc. in a well aligned fashion and grouped if needed.
```c
int _x = 20;
int _y = 30;

int _centerOfScreenX = 120;
int _centerOfScreenY = 230;
```
#
* For macros, use `ALL_CAPS` separated by underscore:
```c
#define LCD_I2C_ADDRESS (0x2A)
```
* To make sure that the macro is not redefined, add a guarding checking:
```c
#ifndef ALOGK
#define ALOGK(message) printf(message)
#else
#warning ALOGK is already defined
#endif
```
or
```c
#ifndef LCD_I2C_ADDRESS
#define LCD_I2C_ADDRESS (0x2A)
#else
#error LCD_I2C_ADDRESS is already defined. Please double-check it.
#endif
```
* If the macro is an operation (not just a literal replacement), enclose it in parentheses:
```c
#define LCD_BACKLIGHT_PWM (0x0F << 12)
#define PROD(x, y) ((x) * (y))
```
Try to avoid confusing & unreadable (with underscores), abbreviated `MACROS`. Set corresponding alias, such as:
```c
#define THIS_FUNCTION __func__
#define THIS_LINE     __LINE__

#define BYTE_ORDER                 __BYTE_ORDER__
#define BYTE_ORDER_LITTLE_ENDIAN   __ORDER_LITTLE_ENDIAN__
#define BYTE_ORDER_BIG_ENDIAN      __ORDER_BIG_ENDIAN__
``` 
#
* For constants, use `camelCase` or `k` followed by `PascalCase`:
```c
const int windowHeight = 400;
const int kWindowWidth = 400;  // (Preferred)
```

* Non-primitive type names should be in `PascalCase`. But, the corresponding `struct` should be in `snake_case`, which makes them compatible with Linux Kernel.
```c
typedef struct qmain_window QMainWindow;
```
or
```c
struct qmain_window {
	...
};
typedef struct qmain_window QMainWindow;
```
While, primitive data types, keep in lower case avoid having longer `snake_case`ish:
```c
typedef unsigned char  byte;
typedef unsigned short uint16;
```
#
* Enum values can either look like macros:
```c
typedef enum process_status {
  PROCESS_STATUS_IDLE,
  PROCESS_STATUS_STOPPED,
  PROCESS_STATUS_RUNNING,
} ProcessStatus;
```
or they can look like constants (preferred):
```c
typedef enum proces_status{
	kProcessStatusIdle,
	kProcessStatusStopped,
	kProcessStatusRunning
} ProcessStatus;
```
Naming template should be like this: either `ENUM_NAME_MEMBER` or `kEnumNameMember`. 
#
* Names of members of structs are `camelCase`:
```c
typedef struct qmain_window CMainWindow;
typedef struct qmain_window {
	int  width;
	int  height;
	bool isFocused;
	bool hasTitleBox;

	GtkWindow *window;
	GtkButton *closeButton;

	void (*onResize)(CMainWindow *self, void *arg);
	void (*onClose) (CMainWindow *self, void *arg);
	void (*show)    (CMainWindow *self, void *arg);
	void (*close)   (CMainWindow *self, void *arg);
};
```
Functions where callbacks should point to, should start with `StructureName` and followed with callback's name in `PascalCase`.
For example:

| Callback | Function name |
| --- | --- |
| `onResize` | `QMainWindowOnResize` |
| `onClose` | `QMainWindowOnClose` |
| `show` | `QMainWindowShow` |
| `close` | `QMainWindowClose` |

#
* Function names should be in `PascalCase`. Opening braces come at the end of the last line for the function declaration rather than on the next line.
```c
bool SerialPortWrite(SerialPort *self, char const* message, size_t length) {
	// Local variables are `_camelCase` and started by underscores.
	bool _messageStatus;
	uint8_t _deviceAddress;

	if (self == NULL || other == NULL || length == 0) {
		return false;
	}
}
```
* For statements that span multiple lines, break after the logical operator and align each line with the start of the first line.
```c
bool QPointIsEqual(CPoint const* self, CPoint const* ref)
	if((self->x == ref->x) &&
	   (self->y == ref->x) &&
	   (self->color == ref->color)) {
		return true;
	}
	return false;
}
```

* For function declarations that span multiple lines, then align subsequent lines with the first parameter.
```c
bool PixelSet(Pixel *self,
			  int x,
			  int y,
			  Color color,
			  float alpha) {
	if(self == NULL) {
		return false;
	}

	pixel->x = x;
	pixel->y = y;
	pixel->color = color;
	pixel->alpha = alpha;
}

int main() {
	// ...
	bool ret = PixelSet(centerPixel,
						centerOfScreenX,
						centerOfScreenY,
						(Color){.red = 150, .blue = 130, .green = 30},
						0.7);
	// ...
}
```
#
### Naming formation
- Function naming formation as per following template: `[<NameSpace>]<Object><ToDo>()`. That is, if namespace is applicable, start with namespace, otherwise, start with object name. Namespace could be in abbreviations such as: `Q`, `C`, `Qt`, `Tf`, `Dsp`, `Gtk`, etc.

| Namespace | Object | Function name (consistent) | Function name (non-consistent) |
| --- | --- | --- | --- |
| `Dsp` | `Filter` | `DspFilterNew()` | `NewDspFilter` or `DspNewFilter` |
| `Dsp` | `Filter` | `DspFilterFree()` | `FreeDspFilter` or `DspFreeFilter` |
| `Dsp` | `Filter` | `DspFilterSetCoefficients` | `SetCoefficientsForDspFilter` |
| `Dsp` | `Filter` | `DspFilterClearBuffer` | `ClearBufferOfDspFilter` |
| n/a | `` | 

- Files naming formation as per following template: `[<namespace_]<object>.<h/c>`.

| Namespace | Object | Filename |
| --- | --- | --- |
| `Q` | `MainWindow` | `q_main_window.h` `q_main_window.c` |
| `Dsp` | `Filter` | `dsp_filter.h` `dsp_filter.c` |
| `Dsp` | `IirFilter` | `dsp_iir_filter.h` `dsp_iir_filter.c` |
| n/a | `Application` | `application.h` `application.c` |

- Data structures and enums naming formation: `[Namespace]Object`. Some examples:

| enum/struct | Namespace | Name | enum/struct name |
| --- | --- | --- | --- |
| `struct` | `q` | `SlideBar` | `typedef struct q_slidebar QSlideBar` |
| `struct` | `dsp` | `AutomaticGainControl` | `typedef struct dsp_automatic_gain_control DspAutomaticGainControl` |
| `enum` | `stm` | `GpioMode` | `typedef enum stm_gpio_mode StmGpioMode` |

#
- Project structure: 
  - Start with `main.c` as an entry point, with minimal code implementations;
  - Add the application logic using `application.h` and `application.c`.
  - Each and every object (structure) should have the declaration part and the definition part (similar to `Qt` OOP). That is: `my_object.h` and `my_object.c`.