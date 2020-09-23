# Tim Tams

- **Category:** Misc
- **Difficulty:** Easy
- **Author:** QUT_WH

When I eat too many Tim Tams, I get rather slow!

**WARNING** You will want to turn down your audio for this one!

**Download:** https://storage.googleapis.com/files.duc.tf/uploads/Clive.wav

**File Hash (SHA256):** 4C1CC12D002956A83E168CA650B776B55AAC36F2131D0DF617BE7D55DBEF93D1

## Writeup

When I played the audio file, I immediately recognized it. I had already completed a challenge featuring the same kind of audio file, and after a long, tedious search, I discovered that this type of file is an SSTV (Slow-Scan Television) signal. SSTV is a method for image transmission over radio signals, which can be decoded using dedicated software such as **MMSSTV**. Let's play this audio file in **Audacity** and output the signal to a virtual audio cable connected to **MMSSTV**. If everything is set up correctly, it should generate this glorious image:

![Glorious Meme from Down Under](./sstv-meme.png)

There's some text suspiciously similar to the flag format `QHPGS{UHZOYR_Z3Z3_1BEQ}`. A simple ROT13, and we get our flag `DUCTF{HUMBLE_M3M3_1ORD}`.