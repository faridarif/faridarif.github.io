---
layout: post
title: "CurtinCTF2024: iHeader"
date: 2024-10-15
categories:
  - CTF WriteUp
---

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/iHeader1.png){:.align-center}

We were given a PNG file that appeared to be **tampered** with.

> *“I received this secret message from a friend, however it looks like someone has tampered with the structure of this image. What could this secret message be?”*

---
## **Initial Analysis**

One of the first steps in analyzing a PNG is to use the tool **`pngcheck`**, which verifies the **integrity** of a PNG file:
```bash
pngcheck -qv header.png
```

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/iHeader2.png){:.align-center}

It showed that the **first chunk must be `IHDR`**, but found `AAAA` instead. This indicated that the **IHDR** chunk type (which identifies as the first critical chunk) was replaced with something else (`AAAA`). Using a hex editor or `xxd`, we confirmed the first few bytes of the chunk type were indeed `41 41 41 41` (ASCII “AAAA”), rather than `49 48 44 52` (ASCII “IHDR”).

---
## **Understanding PNG File Structure**

A valid PNG always starts with **8 specific bytes**:
```python
89 50 4E 47 0D 0A 1A 0A
```
Any deviation means the file likely isn’t a valid PNG. This helps programs identify the file as a PNG. A **chunk** in a PNG file is a self-contained block of data with the following structure:

1. **Length (4 bytes):**
    - Specifies the size of the chunk data.
2. **Chunk Type (4 bytes):**
    - An ASCII value that identifies the type of the chunk (for example `IHDR`, `IDAT`, `IEND`).
3. **Chunk Data (variable length):**
    - Contains the actual data relevant to the chunk.
4. **CRC (4 bytes):**
    - A cyclic redundancy check used to verify the integrity of the chunk.

This modular design allows PNG readers to easily process or skip chunks based on their type.
### **Critical Chunks**

**IHDR** (Image Header)
- **Position:** Always the first chunk after the PNG signature.
- **Purpose:** Contains critical information about the image:
    - **Width** (4 bytes)
    - **Height** (4 bytes)
    - **Bit Depth** (1 byte)
    - **Color Type** (1 byte)
    - **Compression Method** (1 byte)
    - **Filter Method** (1 byte)
    - **Interlace Method** (1 byte)
- **Data Length:** Always 13 bytes.

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/iHeader3.png){:.align-center}

**IDAT** (Image Data)
- **Position:** Follows the IHDR and any ancillary chunks.
- **Purpose:** Contains the actual compressed image data (using the DEFLATE algorithm). Multiple IDAT chunks may exist and need to be concatenated before decompression.

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/iHeader4.png){:.align-center}

### **Ancillary Chunks**

- **`tEXt`, `zTXt`, `iTXt`**: For textual data or metadata.
- **`pHYs`, `gAMA`, `cHRM`**: Various ancillary info (physical pixel dimensions, gamma, color space, etc.).


---

## **Solving The Challenge**

Replace `AAAA` with `IHDR` using hex editor.

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/iHeader5.png){:.align-center}

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/iHeader6.png){:.align-center}

From the `pngcheck` output, it looks like **we’ve successfully restored the first (IHDR) chunk**, but now the second chunk is showing up as `BBBB` with a length of 4292. PNG interprets chunk names in four bytes (each byte is one character). Having a chunk named `BBBB` is causing `pngcheck` to complain because it’s not a recognized (standard) chunk type. For an actual image, the large chunk with 4292 bytes is **very likely** our `IDAT` chunk (which contains the pixel data). Replace the four bytes `42 42 42 42` (ASCII “BBBB”) with `49 44 41 54` (ASCII “IDAT”).

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/iHeader7.png){:.align-center}

Once we replace `BBBB` with `IDAT`, try to re-running `pngcheck`. If the rest of the file structure is intact, it should parse correctly and display a valid PNG.

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/iHeader8.png){:.align-center}

We can see that there’s an **`iTXt`** chunk near the end. The `iTXt` is used to store textual data in a PNG, which is the flag (open the image to get the flag).

---
