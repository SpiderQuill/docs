![Quill OS logo](https://raw.githubusercontent.com/PorQ-Pine/branding/refs/heads/main/quillos-banner.svg)
---

### Contact and general information
- You are welcome to come and join us at our [Matrix space](https://matrix.to/#/#porq-pine:matrix.org)!
- You can also watch a video demo of the OS [here](https://www.youtube.com/watch?v=HORhEmINF68).

# Useage

**DISCLAIMER:** Quill OS for the PineNote is in its early stages of development. Expect rough edges, missing features (e.g., screen rotation), and potential bugs. This installation process is complex and carries risks, including the possibility of data loss (if you don't follow it correctly) or a temporarily non-functional device. Proceed with caution and at your own risk.

Quill OS' goal is to provide an experience for the PineNote which is:

- Refined, so special software written for eink, adjusted for eink
- Easy to use, without a external keyboard, just as a regular android eink tablet, but still advanced tiling for users who want it
- Recoverable in field: Linux users are tinkerers, things break because of us and without us, Quill OS should be recoverable to usable state without another machine.
- Reasonable security but hackable, home directory encryption by default for example, no limitations to what you do to it
- Easy to install/reproductible, even from scratch by compiling, everything automated
- All the software written by the community for the pinenote (HRDL kernel, etc) is managed by us for you, no need for you to be a linux expert to install it and use it

### Current state
---
Usable for advanced users, but there are many major and minor issues to be solved. Please see the [Issues](https://github.com/PorQ-Pine/quillstrap/issues) If you have any.

### Installation & Updating
---
Please Refer to these Documents when it comes to Installing and Updating things. There are a lot of small patches and tweaks being made that can easily be broken by normal update methods.
- [Installation Docs](/public/install_from_scratch.md)
- [Updating Docs](/public/update_from_source.md)

### Legacy
---
There is old [Quill OS (Formerly InkBox)](https://github.com/Quill-OS/quill) Which was almost fully created by [Nicolas](https://github.com/tux-linux) and [Szybet](https://github.com/Szybet) and other contributors along the way.

That OS was killed by the manufacturer.

This organisation, named PorQ-Pine, is a play on **Por**ting **Q**uill to the **Pine**(Note). This project is a complete rewrite of the origional project. Thank you to Pine64 for providing hardware and thank you to the pine community for providing building blocks in software that this projects uses.

### Design principles
---
Because we learn from our mistakes, (And the Pinenote user target is different from Kobo devices)

1. Prioritizes Rust.
2. It should be easily reproductible via [quillstrap](https://github.com/PorQ-Pine/quillstrap/tree/main)
3. a) Hackable, but b) Recoverable without needing another machine. These 2 points conflict with each other, usually, so balancing them is needed
4. Reasonable security (So home directory encryption by default, but don't conflict with point 3)
5. Choosing solutions which are easily maintanable and follow current standards of doing things, avoid being hacky as much as possible
