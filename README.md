# didiff
Dicom diff - a diff-like tool for comparing DICOM files

Author: Jiří Keller, M.D., Ph.D.

This SW is a result of the research funded by the project Nr. LO1611 with a financial support from the MEYS under the NPU I program.

For more detailed manual, please refer to [readme.txt](readme.txt) file.

# Brief introdution ...
## ... to DICOM
DICOM is a well established medical image communication and storage format.
From the user's perspective, each DICOM file consists of a header and medical
data (typically image such as CT or MRI, but can be anything from spectroscopic
curve to endoscopic screenshots or photos from your vacation). In reality,
there is no difference between header and data in DICOM standard, as it
contains only key-value pairs (image data is just another "value"). In DICOM
world, the "keys" are called tags and they are referenced by (Group,Element)
pair as there is makes handling more than 2000 items in the DICOM 3 standard
easier. Each Group and Element is a hexadecimal two bytes number. The whole
list is listed in part PS3.6 of the DICOM standard (caled Data Dictionary) - see
(http://dicom.nema.org/medical/dicom/current/output/html/part06.html).
Standard tags have even Group numbers, odd Group numbers are reserved for
so called "private" tags, which are typically vendor-specific and may contain
a lot of interesting information, unfortunately in most of the cases undocumen-
ted. In Standard DICOM Dictionary, tags such as patient name (0010,0010),
sex (0010,0040), age, weight, medical device which produced image and many
others are defined. And, by the way, your image is stored in PixelData tag
which is (7fe0,0010).

## ... to didiff
Dicomdiff (or didiff) is tool inspired by GNU diff, which compares two files
line by line. Our tool compares DICOM tags between two DICOM files or a DICOM
file and ASCII dump (see [readme.txt](readme.txt) for details).

There are three basic output modes:
1. basic diff-like output (values from each files on separate lines)
2. "first file value" and "the corresponding value" with tag and one value per line
3. CSV output with tag and both values on one line

CSV (comma separated values) format can be very handy as it can be easily
processed using any Office package or an automatic script (eg. awk).
