---
title: Font, Charset & Encoding
date: 2026-03-11 13:05:00
categories: Encoding
---

For text shown on the internet (blogs, instant messages, etc.), have you ever wondered how it is displayed?

It relates to three basic concepts, from frontend to backend: **Font**, **Character set (charset)**, and **Encoding**.

## Font

On digital devices, text is ultimately rendered into images (a matrix of pixels). A **font** describes how characters are drawn.

A font typically stores outlines as vector graphics (often Bézier curves), so glyphs can be drawn at many sizes and in different styles.

Common font file types are [OpenType (OTF)](https://en.wikipedia.org/wiki/OpenType), [TrueType (TTF)](https://en.wikipedia.org/wiki/TrueType), and [WOFF/WOFF2](https://en.wikipedia.org/wiki/Web_Open_Font_Format).

## Character set (charset)

How does a program know which glyph in the font to draw?

You need a mapping between **characters** and numbers. Take Unicode as an example, each character is assigned a **code point** (for example, `你` is `U+4F60`, and `A` is `U+0041`).

Fonts don’t store "characters" directly; they store **glyphs** (shapes). A font’s **cmap** table maps **code points → glyphs**. Not every glyph necessarily has its own code point (for example, ligatures and stylistic alternates).

A charset is not a physical thing; it’s a standard/specification that everyone follows. The most widely used character set today is [Unicode](https://en.wikipedia.org/wiki/Unicode). Older/legacy character sets include *ASCII* and many regional standards.

## Encoding

Computers store and transmit **bytes**, so we need an **encoding**: a rule for turning code points into bytes (and back). Without knowing the encoding, a byte sequence can’t be interpreted correctly.

Some encodings are fixed-length (for example, **UTF-32** uses 4 bytes per code point). Others are variable-length: **UTF-8** uses 1–4 bytes, and **UTF-16** uses 2 or 4 bytes (surrogate pairs for code points above `U+FFFF`). Variable-length encodings are efficient for common text, especially when most characters are in the ASCII range.

The most common encoding on the web is [UTF-8](https://en.wikipedia.org/wiki/UTF-8), a Unicode-based variable-length encoding. For example, `你` (`U+4F60`) is encoded in UTF-8 as the bytes `E4 BD A0` (hex).

---

This article demonstrates the three basic concepts behind text rendering. In real systems there are more steps, like shaping, layout/positioning, and rasterization.