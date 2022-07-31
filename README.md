# Bitmap Font CSS

I love bitmap fonts. There are some great web-compatible bitmap fonts available. Here are some of my favourites:

 - https://int10h.org/oldschool-pc-fonts/
 - https://files.ax86.net/terminus-ttf/ (**TODO:** I love Terminus but this TTF conversion looks bad if you zoom in - One day I'll make my own, better conversion)
 - https://unifoundry.com/unifont/

Unfortunately, web browsers aren't built to handle bitmap fonts properly - the above fonts only work on the web when converted into a vector font format.

Because browsers don't know they're supposed to be bitmap fonts, they end up rendered poorly, often with fuzzy edges (because the vector edges don't always lie neatly on pixel boundaries, etc.)

There are also issues with mobile browsers deciding to arbitrarily adjust your font sizes, sometimes on a per-glyph basis.

This repo documents my attempts to wrangle web browsers in to rendering them nicely.

It's a git repo as opposed to a blog post because the workarounds will inevitably change over time, and it's currently imperfect, so I welcome discussion on how it could be improved.

**TODO:** A simple demo webpage, to let you test different options.

**TODO:** Some demonstration screenshots.

**TODO:** Create the ultimate workaround - canvas-rendered true-bitmap fonts with a transparent text overlay (so you can still select text, etc.)

## The Basics

I'm using the IBM VGA 8x16 font here as an example here. It's 16 pixels high, so I set the `font-size` accordingly. If you want your fonts to be larger, I recommend sticking to integer multiples of the original pixel size.

The `line-height` property is also important. I set it to `1` so that ANSI art displays as intended - with no vertical gaps between the line-drawing characters. However for larger bodies of text, you might want a larger `line-height` for legibility reasons - if you do, I recommend sticking to integer pixel values.

```css
@font-face {
  font-family: "ibmpc";
  src: url("./WebPlus_IBM_VGA_8x16.woff") format("woff");
}

body {
  font-family: "ibmpc", monospace;
  font-size: 16px;
  line-height: 1;
}
```

## Disable Font Size "Adjusting"

Mobile browsers have decided that web developers can't be trusted to set good font sizes, and must be overruled. They're not wrong, but such adjustments can really ruin our bitmap font aesthetic, so these rules remind the browser to do as it's told:

```css
body {
  text-size-adjust: none;
  -webkit-text-size-adjust: none;
}
```

## Disable Anti-Aliasing

Subpixel (or even whole-pixel) anti-aliasing makes bitmap font edges look fuzzy, even without scaling involved. The quick way to disable it is as follows:

```css
body {
  -webkit-font-smoothing: none;
  -moz-osx-font-smoothing: grayscale;
  font-smooth: never;
}
```

(Unfortunately, I think this only works on OSX/iOS, for now. See https://caniuse.com/font-smooth)

However, there are some cases where no anti-aliasing makes things look worse. For example, when a font is scaled by a non-integer ratio on a low-DPI display. As a workaround, I use CSS media queries to detect when the viewport is scaled by a non-integer amount:

```css
@media
  (resolution: 1dppx),
  (resolution: 2dppx),
  (resolution: 3dppx),
  (resolution: 4dppx)
{
  body {
    -moz-osx-font-smoothing: grayscale;
    font-smooth: never;
  }
}
```

There's one big caveat - this does not work on Safari. Firstly, Safari doesn't support the `dppx` unit, but that has workarounds such as `-webkit-device-pixel-ratio`. The real issue with Safari is that it "lies" and always reports the same pixel ratio, even when the viewport has been scaled. So on Safari, you get to pick between always-on and always-off. I prefer to leave anti-aliasing on, because it still looks *alright* on desktop Safari, wereas no anti-aliasing looks terrible on my iPhone SE.
