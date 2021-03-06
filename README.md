# pyzbar

[![Python Versions](https://img.shields.io/badge/python-2.7%2C%203.4%2C%203.5%2C%203.6-blue.svg)](https://github.com/NaturalHistoryMuseum/pyzbar)
[![PyPI version](https://badge.fury.io/py/pyzbar.svg)](https://pypi.python.org/pypi/pyzbar/)
[![Travis status](https://travis-ci.org/NaturalHistoryMuseum/pyzbar.svg?branch=master)](https://travis-ci.org/NaturalHistoryMuseum/pyzbar)
[![Coverage Status](https://coveralls.io/repos/github/NaturalHistoryMuseum/pyzbar/badge.svg?branch=master)](https://coveralls.io/github/NaturalHistoryMuseum/pyzbar?branch=master)

Read one-dimensional barcodes and QR codes from Python 2 and 3 using the
[zbar](http://zbar.sourceforge.net/) library.
Works with PIL / Pillow images, OpenCV / numpy `ndarray`s, and raw bytes.

The older [zbar](https://sourceforge.net/p/zbar/code/ci/default/tree/python/)
package is stuck in Python 2.x-land.
The [zbarlight](https://github.com/Polyconseil/zbarlight/) package doesn't
provide support for Windows and depends upon Pillow.
This pure-Python package uses `ctypes` to bring `zbar` to Python 2.7 and
Python 3.4 or greater.

## Installation

The `zbar` `DLL`s are included with the Windows Python wheels.
On other operating systems, you will need to install the `zbar` shared library.

Mac OS X:

```
brew install zbar
```

Linux:

```
sudo apt-get install libzbar0
```

Install this Python wrapper; use the second form to install dependencies of
the command-line scripts:

```
pip install pyzbar
pip install pyzbar[scripts]
```

## Example usage

The `decode` function accepts instances of `PIL.Image`.

```
>>> from pyzbar.pyzbar import decode
>>> from PIL import Image
>>> decode(Image.open('pyzbar/tests/code128.png'))
[Decoded(data=b'Foramenifera', type='CODE128', rect=Rect(left=37, top=550, width=324, height=76)),
Decoded(data=b'Rana temporaria', type='CODE128', rect=Rect(left=4, top=0, width=390, height=76))]
```

It also accepts instances of `numpy.ndarray`, which might come from loading
images using [OpenCV](http://opencv.org/).

```
>>> import cv2
>>> decode(cv2.imread('pyzbar/tests/code128.png'))
[Decoded(data=b'Foramenifera', type='CODE128', rect=Rect(left=37, top=550, width=324, height=76)),
Decoded(data=b'Rana temporaria', type='CODE128', rect=Rect(left=4, top=0, width=390, height=76))]
```

You can also provide a tuple `(pixels, width, height)`, where the image data
is eight bits-per-pixel.

```
>>> image = cv2.imread('pyzbar/tests/code128.png')
>>> height, width = image.shape[:2]

>>> # 8 bpp by considering just the blue channel
>>> decode((image[:, :, 0].astype('uint8').tobytes(), width, height))
[Decoded(data=b'Foramenifera', type='CODE128', rect=Rect(left=37, top=550, width=324, height=76)),
Decoded(data=b'Rana temporaria', type='CODE128', rect=Rect(left=4, top=0, width=390, height=76))]

>>> # 8 bpp by converting image to greyscale
>>> grey = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
>>> decode((grey.tobytes(), width, height))
[Decoded(data=b'Foramenifera', type='CODE128', rect=Rect(left=37, top=550, width=324, height=76)),
Decoded(data=b'Rana temporaria', type='CODE128', rect=Rect(left=4, top=0, width=390, height=76))]

>>> # If you don't provide 8 bpp
>>> decode((image.tobytes(), width, height))
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/Users/lawh/projects/pyzbar/pyzbar/pyzbar.py", line 102, in decode
    raise PyZbarError('Unsupported bits-per-pixel [{0}]'.format(bpp))
pyzbar.pyzbar_error.PyZbarError: Unsupported bits-per-pixel [24]
```

The default behaviour is to decode all symbol types. You can look for just your
symbol types

```
>>> from pyzbar.pyzbar import ZBarSymbol
>>> # Look for just qrcode
>>> decode(Image.open('pyzbar/tests/qrcode.png'), symbols=[ZBarSymbol.QRCODE])
[Decoded(data=b'Thalassiodracon', type='QRCODE', rect=Rect(left=27, top=27, width=145, height=145))]

>>> # If we look for just code128, the qrcodes in the image will not be detected
>>> decode(Image.open('pyzbar/tests/qrcode.png'), symbols=[ZBarSymbol.CODE128])
[]
```

## Windows error message
If you see an ugly `ImportError` when importing `pyzbar` on Windows you will
most likely need the
[Visual C++ Redistributable Packages for Visual Studio 2013](https://www.microsoft.com/en-US/download/details.aspx?id=40784).
Install `vcredist_x64.exe` if using 64-bit Python, `vcredist_x86.exe` if using
32-bit Python.

## Contributors

* Alex (@globophobe) - first implementation of barcode locations

## License

`pyzbar` is distributed under the MIT license (see `LICENCE.txt`).
The `zbar` shared library is distributed under the GNU Lesser General Public
License, version 2.1 (see `zbar-LICENCE.txt`).
