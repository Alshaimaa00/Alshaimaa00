#!/usr/bin/env python3
"""Builds header.svg (1400x400, animated) for the GitHub profile.

Usage:
    python3 build_header.py                 -> header.svg with an empty photo slot
    python3 build_header.py my_photo.png    -> header.svg with the photo embedded

The photo is embedded as base64 inside the SVG, because GitHub does not load
external files referenced from an SVG that is shown through <img>.
"""
import base64
import mimetypes
import sys
from pathlib import Path

W, H = 1400, 400
NAME = "ALshaimaa Alamer"
TITLE = "UI/UX Designer &amp; Developer"
TAGLINE = "Designing simple, useful apps"

# Photo slot (right side)
CX, CY, R = 1130, 200, 135


def photo_block(photo_path):
    if not photo_path:
        # Empty slot: dashed ring, nothing drawn inside.
        return f"""
  <circle cx="{CX}" cy="{CY}" r="{R}" fill="rgba(255,255,255,0.06)"
          stroke="#ffffff" stroke-opacity="0.55" stroke-width="3" stroke-dasharray="10 12"/>
"""
    mime = mimetypes.guess_type(photo_path)[0] or "image/png"
    data = base64.b64encode(Path(photo_path).read_bytes()).decode()
    return f"""
  <clipPath id="photoClip"><circle cx="{CX}" cy="{CY}" r="{R}"/></clipPath>
  <image href="data:{mime};base64,{data}" x="{CX - R}" y="{CY - R}"
         width="{2 * R}" height="{2 * R}" preserveAspectRatio="xMidYMid slice"
         clip-path="url(#photoClip)"/>
  <circle cx="{CX}" cy="{CY}" r="{R}" fill="none" stroke="#ffffff" stroke-width="5"/>
"""


def build(photo_path=None):
    return f"""<svg xmlns="http://www.w3.org/2000/svg" width="{W}" height="{H}"
     viewBox="0 0 {W} {H}" role="img" aria-label="{NAME}">
  <defs>
    <linearGradient id="bg" x1="0" y1="0" x2="1" y2="1">
      <stop offset="0%" stop-color="#1b1446">
        <animate attributeName="stop-color" values="#1b1446;#3a1c71;#1b1446" dur="10s" repeatCount="indefinite"/>
      </stop>
      <stop offset="55%" stop-color="#6a2c8f">
        <animate attributeName="stop-color" values="#6a2c8f;#b24592;#6a2c8f" dur="10s" repeatCount="indefinite"/>
      </stop>
      <stop offset="100%" stop-color="#f15f79">
        <animate attributeName="stop-color" values="#f15f79;#ff9966;#f15f79" dur="10s" repeatCount="indefinite"/>
      </stop>
    </linearGradient>
    <radialGradient id="glow" cx="0.5" cy="0.5" r="0.5">
      <stop offset="0%" stop-color="#ffffff" stop-opacity="0.35"/>
      <stop offset="100%" stop-color="#ffffff" stop-opacity="0"/>
    </radialGradient>
  </defs>

  <style>
    .name   {{ font: 800 78px 'Segoe UI', Helvetica, Arial, sans-serif; fill: #fff; }}
    .title  {{ font: 600 32px 'Segoe UI', Helvetica, Arial, sans-serif; fill: #ffe3ec; }}
    .tag    {{ font: 400 24px 'Segoe UI', Helvetica, Arial, sans-serif; fill: #ffffff; fill-opacity: .8; }}
    .fadeup {{ opacity: 0; animation: fadeup 1s ease-out forwards; }}
    .d1 {{ animation-delay: .2s; }} .d2 {{ animation-delay: .6s; }} .d3 {{ animation-delay: 1s; }}
    @keyframes fadeup {{ from {{ opacity: 0; transform: translateY(18px); }} to {{ opacity: 1; transform: translateY(0); }} }}
    .float1 {{ animation: float 7s ease-in-out infinite; }}
    .float2 {{ animation: float 9s ease-in-out infinite reverse; }}
    .float3 {{ animation: float 11s ease-in-out infinite; }}
    @keyframes float {{ 0%,100% {{ transform: translateY(0); }} 50% {{ transform: translateY(-22px); }} }}
    .pulse {{ transform-origin: {CX}px {CY}px; animation: pulse 4s ease-in-out infinite; }}
    @keyframes pulse {{ 0%,100% {{ transform: scale(1); opacity: .9; }} 50% {{ transform: scale(1.12); opacity: .5; }} }}
  </style>

  <!-- background fills the whole canvas: no side margins -->
  <rect width="{W}" height="{H}" fill="url(#bg)"/>

  <!-- floating shapes -->
  <g class="float1"><circle cx="90" cy="320" r="46" fill="#ffffff" fill-opacity="0.10"/></g>
  <g class="float2"><circle cx="720" cy="70" r="30" fill="#ffffff" fill-opacity="0.12"/></g>
  <g class="float3"><rect x="640" y="300" width="64" height="64" rx="18" fill="#ffffff" fill-opacity="0.10" transform="rotate(18 672 332)"/></g>
  <g class="float1"><circle cx="1330" cy="60" r="22" fill="#ffffff" fill-opacity="0.14"/></g>
  <g class="float2"><circle cx="1290" cy="340" r="38" fill="#ffffff" fill-opacity="0.10"/></g>

  <!-- glow behind the photo -->
  <circle class="pulse" cx="{CX}" cy="{CY}" r="{R + 45}" fill="url(#glow)"/>
  {photo_block(photo_path)}

  <!-- text -->
  <g class="fadeup d1"><text class="name" x="90" y="170">{NAME}</text></g>
  <g class="fadeup d2"><text class="title" x="94" y="225">{TITLE}</text></g>
  <g class="fadeup d3"><text class="tag" x="94" y="275">{TAGLINE}</text></g>
</svg>
"""


if __name__ == "__main__":
    photo = sys.argv[1] if len(sys.argv) > 1 else None
    out = Path(__file__).with_name("header.svg")
    out.write_text(build(photo), encoding="utf-8")
    print(f"wrote {out} ({W}x{H}){' with photo' if photo else ' (empty photo slot)'}")
