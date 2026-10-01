<h1 align="center">InXpoze</h1>

<p align="center">
  <strong>A photo privacy X-ray for Windows that shows what an image gives away before you share it, and exports a clean copy.</strong>
</p>

<p align="center">
  <img alt="platform" src="https://img.shields.io/badge/Windows-10%20%7C%2011-0078d4">
  <img alt="dependencies" src="https://img.shields.io/badge/dependencies-none-brightgreen">
  <img alt="telemetry" src="https://img.shields.io/badge/telemetry-none-555">
  <img alt="offline" src="https://img.shields.io/badge/network-blocked%20by%20default-ffc233">
</p>

---

## What it is

A photograph carries far more than the picture. Your phone writes the
coordinates it was taken at, accurate to a few metres. Your camera writes a
serial number that links every photo you have ever posted to the same device.
The file keeps a small hidden thumbnail that no viewer displays, and if you
cropped the photo afterwards, that thumbnail is often still the uncropped
original. Some phones append a few seconds of video, with audio, after the end
of the image.

None of that is visible. All of it travels with the file.

InXpoze reads it. It parses the container byte by byte rather than handing the
file to an imaging library, so a damaged photo is still readable and every
block's exact position is known. Then it reads the **picture** too, because
stripping metadata does nothing about a house number on a wall, a number plate
in a driveway, or an inbox open on a monitor in the background.

Everything is scored out of 100 and explained in plain English. Not "EXIF
GPSLatitude present", but:

> From this photo alone, someone could work out exactly where this was taken,
> in Newtown, New South Wales, what you cropped out of the picture, a serial
> number that links this photo to every other photo you have posted from the
> same camera, and when you were there.

You then cover what you want covered and export a new file. The original is
never modified, and InXpoze re-reads the exported copy afterwards to confirm it
really is clean.

**It all runs on your machine.** Network access is blocked inside the program
at process level, not merely unused, and you can watch it fail from the About
screen.

---

## Download

Grab the latest **`InXpozeInstaller.exe`** from the
[Releases page](https://github.com/FadyAzzi/InXpoze/releases) or
[Direct Link](https://github.com/FadyAzzi/InXpoze/releases/latest/download/InXpozeInstaller.exe).

Run it and follow the wizard.

**Requirements:** Windows 10 or 11, 64-bit. Nothing else.

Nothing else to install. Python, every library and all detection models are
inside the installer. There is no model to download on first run, no separate
OCR engine to set up, and no internet connection needed at any point.

---

## Features

### Metadata X-ray

Everything hidden in the file, grouped by risk and explained in plain English.

**High risk**

- **GPS coordinates**, altitude and the satellite timestamp, with the place
  name resolved **offline** from a bundled cities database. Asking an online
  geocoder "where is this?" would leak the exact thing the app protects, so
  InXpoze never does.
- **The hidden thumbnail, compared against the photo you can see.** If they
  differ, you cropped the picture and your editor did not update the
  thumbnail, so the original framing is still in the file. The comparison runs
  on colour, not brightness, because a red front door and a blue front door
  are not the same scene.
- **Data after the end of the image.** A JPEG ends at `FFD9`; anything after
  that is invisible in every viewer and still gets sent. Usually a motion
  photo video or a depth map, occasionally a whole second file.
- **Camera and lens serial numbers**, which tie every photo from that device
  together across every account and every site, even where you used a
  different name.
- **Names of people** written in by a photo library that recognised their
  faces, read from the Metadata Working Group, Microsoft and IPTC region
  schemas.

**Also read**

EXIF, XMP, IPTC, PNG text chunks, maker notes, Multi-Picture Format indexes
and C2PA content credentials. Timestamps, time zones, owner and copyright
fields, software versions, and the exposure settings that give nothing away.

### Visual scan

Stripping metadata does not touch any of this.

- **Faces**, detected with YuNet, so you can cover the ones that are not the
  point of the photo.
- **Readable text**, via on-device OCR. House numbers, street signs, names on
  parcels, whiteboards, name badges and documents.
- **Number plates**, matched against the plate formats for Australia, the UK,
  the US or Europe. A pattern loose enough to match everywhere matches too
  much, so the region is a setting.
- **Screens and devices** in shot, plus people and vehicles.
- **What leaks out of screenshots**: email addresses, phone numbers, payment
  card numbers (validated with the Luhn check, so a reference number is not
  reported as a card), IBANs, API keys and access tokens, URLs carrying a
  session, Windows user paths and dates of birth.

### Risk score

A score out of 100 with a verdict of **Safe**, **Caution**, **Risky** or
**Very risky**, and a panel that shows the arithmetic on request.

Two rules keep it honest. A finding that is true of every photograph by
construction, like having pixel dimensions, contributes nothing, because
something present in every case cannot distinguish between cases. And repeated
findings in one category have diminishing returns, so a heavily tagged stock
photo cannot out-score a picture of your front door with the GPS attached.

### Redaction

- **Solid block**, **heavy pixelate** or **strong blur**, applied to the pixels
  and re-encoded. Never an overlay: an exported file with the original pixels
  under a black rectangle looks perfect on screen and protects nobody.
- **Anything containing text is always covered with a solid block**, whatever
  style you picked. Blur and pixelation are reversible on text, because the set
  of possible originals is small enough to search: a four-digit house number
  has ten thousand candidates, and each can be rendered, blurred the same way
  and compared. A face has no such list, so pixelation is a real choice there.
- **Draw your own box** over anything the scan missed, with full undo and redo.
- **Share presets** for the way you are actually sharing it: Social media,
  Marketplace listing, Dating profile, Evidence and legal, or strip metadata
  only.

Cover something and the canvas switches straight to the redacted version, so
you see what the photo will look like rather than a box over it.

### Export

- Always writes a **new file**. Your original is never modified, and its
  SHA-256 is shown before and after so you can confirm that.
- Never overwrites. Exporting twice gives you two files.
- Removes **every** metadata block, including the embedded thumbnail, and fully
  re-encodes the image.
- **Checks the file it just wrote.** InXpoze re-reads the exported copy off
  disk, with its own parsers plus a raw byte sweep, and tells you what it
  actually contains. A library upgrade that started emitting something by
  default would be invisible to a check that simply asked the writer what it
  had done.

### Batch

Scan a whole folder and strip it in one action. Batch also finds the thing no
single-photo tool can:

> **Likely home or workplace near Newtown, New South Wales, AU:** 23 photos
> taken here on 14 different days. That is almost certainly somewhere you live,
> work or go every week.

One geotagged photo says where it was taken. Forty say where you live and when
you are usually out. A location is only called a likely home when photos from
several **different** days support it, because twenty photos from one afternoon
are an event, not a residence.

### Reports

Export the findings as **HTML** or **PDF** for a client, a case file, or a
record of what was found before an image was published. The report contains
findings and no image data; the file hash identifies the subject instead.

---

## Supported formats

| In | Out |
| --- | --- |
| JPEG, PNG, HEIC / HEIF, WebP, TIFF, BMP | JPEG, PNG, WebP |

---

## Privacy

InXpoze analyses photos on your machine and does not upload them. It collects
no telemetry, and there is no account.

Network access is **blocked inside the program itself**, at process level, by a
kill switch armed before anything else at startup. The About screen has a
**Test it now** button that opens a real outbound connection and shows InXpoze
refusing it.

Two features can use a connection, and each can be switched off:

| Feature | Contacts | Sends | Default |
| --- | --- | --- | --- |
| Update check | GitHub | nothing about you or your photos | on |
| Map view | OpenStreetMap | the coordinates you asked to see | off |

Those are the only two sanctioned exceptions in the codebase, and both are
greppable as `allow_temporarily`. There is no cloud service, no account and
no AI model: every finding InXpoze reports is produced by code in this
repository.

Settings and the log live under `%LOCALAPPDATA%\InXpoze` and are removed when
you uninstall. **The log records what InXpoze did, never what it saw**: file
names and counts, never coordinates, recognised text or any image data.

---

## A caution worth reading

InXpoze reports what it can detect, and it will miss things. Automatic
detection of faces, text, plates and objects is imperfect, metadata can be
stored in ways it does not recognise, and no program can tell you everything a
particular person might work out from a particular photograph.

Treat the findings as a reason to look more closely, not as proof that an image
is safe to publish. Check the exported copy before you share it.

---

## Part of InCyberField

InXpoze is one of the **InCyberField** tools, a suite of security and privacy
applications for Windows. Each installs as a single application, runs locally,
and carries its own terms.

---

## Author

**Fady Azzi**

Copyright © 2026 Fady Azzi. All rights reserved.

---

## NOTE

Some anti-virus products or EDRs might block the download of the exe file, as it
is not yet signed with a code-signing certificate. You can download the zip file
as a workaround.

Please download the latest exe or zip version from the "Releases" section on the
right, or use the direct links below.

Direct link to the exe is:
https://github.com/FadyAzzi/InXpoze/releases/latest/download/InXpozeInstaller.exe

Direct link to the zip is:
https://github.com/FadyAzzi/InXpoze/releases/latest/download/InXpozeInstaller.zip
