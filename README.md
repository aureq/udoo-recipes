udoo-recipes
============

This repository contains few recipes to help people create their own little distro for specific hardware.

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="http://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a> <span xmlns:dct="http://purl.org/dc/terms/" href="http://purl.org/dc/dcmitype/Text" property="dct:title" rel="dct:type">Udoo recipes</span> by <a xmlns:cc="http://creativecommons.org/ns#" href="http://au.linkedin.com/in/aurelienrequiem/" property="cc:attributionName" rel="cc:attributionURL">Aurélien Requiem</a> is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>. Based on a work at <a xmlns:dct="http://purl.org/dc/terms/" href="https://github.com/aureq/udoo-recipes" rel="dct:source">https://github.com/aureq/udoo-recipes</a>.

# Platform #

Supported platforms are

- Udoo Quad
- Wandboard Quad

# Ackowledgements #

- [Stéphan Rafin](http://stephan-rafin.net/blog/) for implementing the imx6 hardware support in Xbmc and the Yocto image.
- The Udoo community.
- Anyone who was kind enough to publish their work on Internet.

# Environment #
- Debian 7, x64, 4 CPUs, 4GB ram

# Known Issues #
- CEC is not working due to a wiring issue on the udoo board (as explained by [Stéphan Rafin](http://stephan-rafin.net/blog/)).
- Udoo doesn't power-off when Xbmc exits
- Reboot fails and requires a power cycle

# To-do #
- Document wifi network set up
- Document the Wiimote support
- Document MySQL support (set up and credentials)

# Recipes #
1. [Rootfs recipe](01-rootfs-recipe.md)
2. [uImage recipe](02-uimage-recipe.md)
3. [XBMC recipe](03-xbmc-recipe.md)
