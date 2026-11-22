---
name: egyptian-tts-narration
description: Write and produce Egyptian colloquial (Cairene) Arabic voiceover with an ElevenLabs voice clone, and fix mispronounced words by test-and-splice without regenerating approved audio.
---

# Egyptian colloquial TTS narration (ElevenLabs voice clone)

## When to use
- Turning a book/script text into a spoken-spelling script for Egyptian (Cairene) narration.
- Generating narration segments with the owner's ElevenLabs Professional Voice Clone.
- Fixing a word the owner says is mispronounced in an already-approved segment.

The owner's ear is the only judge. All rules below came from his listening reviews.

## Voice settings (fixed)
| Setting | Value |
|---|---|
| Voice | Owner's own Professional Voice Clone |
| Model | `eleven_v4` |
| Language | **Auto**. Never force `ar`: it pushes Modern Standard Arabic |
| Stability | 0.5 |
| Similarity | 0.75 (on the website: lowest clone strength) |

Report the credit cost of **every** generation, including tiny tests.

## Spelling rules: write it the way it is said ("كما يُنطق")
1. Arabic letters only, spelled the way a Cairene says the word. **No IPA**: the model read IPA `j` (ي) as ج, and IPA was unreliable overall. Old `/…/` lexicon entries are only a guide to the sound, never output.
2. **No full tashkeel**: it pushes MSA. Add diacritics only to the one word that needs them (e.g. يِلْؤُطْ، تِتْغِسِل، نَصّ).
3. **Qaf**: in purely colloquial words, write ق as the hamza that is actually said. Words of Standard-Arabic origin **keep ق**. Do not over-convert (the owner flagged this: "بتأفور في الصح لحد ما يبقى غلط").
4. **Tha**: educated words keep ث (ثواني، واثق). تواني / واسِأ were wrong.
5. **Context decides** vowels and meaning. Read and understand the sentence before respelling. Never respell mechanically.
6. **Tool and brand names in English letters** (ChatGPT, Claude, Gemini). Written in Arabic, ج is read as a Cairene g.
7. If a pronoun suffix keeps coming out wrong, split it off: بتدخّل له.
8. Some words only come out right in a **very short phrase**; inside a long sentence the model reverts (e.g. «كل ما تِئِلّ،»). Generate such a word in a short phrase.
9. Conjunction و before an imperative is written and said: «اسأله سؤال إجابته طويلة، ووَأَّفْ الرد في نُصّو».

### Approved words (confirmed by the owner)
| Group | Book text → spoken spelling |
|---|---|
| Qaf → hamza (colloquial) | قاعد→آعِد · فوق→فوء / فوؤ · بيقرا→بيِئْرا · بيقراه→بيِئْراه · قال→آل · قعد→أعد · قبله→أبله · يلقط→يِلْؤُطْ · بيلقط→بيِلْؤُطْ · طريقة→طريأة · يقرّب→يِأرّب |
| Keep ق (Standard-Arabic origin) | اقتراح / يقترح / الاقتراحات · قاموس · قماش→قُماش · مقالات · ترقية / اترقّى · مقنع→مُقنِع · القطعة · توقّع · قرارات · ينقل→يِنقِل |
| Keep ث | ثواني · واثق |
| Context-sensitive | شبه بعضه→شَبَه بعضه (similar) · تقل→تِئِلّ (becomes less, not ثقل; say it in «كل ما تِئِلّ،») · نصه→نُصّو (its half) · نص (text)→نَصّ vs نُصّ (half) · النصية→النَّصّية · وقف (imperative)→وَأَّفْ, with و: ووَأَّفْ · وقف (past)→وِئِف · رجع (past)→رِجِع · ما حفظش→ما حَفَظْش · بتدخله→بتدخّل له · يطلّع (transitive) vs يطلع (goes up) · تتغسل→تِتْغِسِل · تغسل (quoted)→«تُغسل» |
| English letters | ChatGPT · Claude · Gemini · Large Language Model · Next Token Prediction · Token · Temperature |

Shadda on doubled verbs helps: يجرّب، يدوّر، اتدرّب، يزوّد، يكرّر، اتغيّر، كمّل.

**Lexicon.** `eps/lexicon.json` holds entries `{word, ctx, out, ok, ep, note}` and a `rules` list. Before writing a script, apply entries marked `ok: true` (Arabic spellings). Read each sentence first, because meaning decides. Every word the owner confirms goes into the lexicon right away with `ok: true`, its context and what failed.

## Script format
- Start from the plain book text. Respell only what the rules require; leave every other word as written.
- Put `[pause]` after each sentence inside a paragraph, and after a colon that introduces speech or a list («بيسأل نفسه: [pause] …»).
- Separate paragraphs with a blank line; no `[pause]` at a paragraph's end.
- **Segments**: about 20–30 s each, made of **whole paragraphs**. Never cut a sentence across segments. Store as a JSON list of strings (`final_scripts/epN_segments.json`); audio files `01.mp3`, `02.mp3`…
- Keep a review sheet with each segment's time range in the assembled episode so the owner can say "segment 8, the word تقل".

## Review and fix workflow
1. **Generate one segment at a time** with the fixed settings. Report credits used.
2. The owner names the **segment and the word**. Change only what he names. If unsure what he means or whether to spend credits, **ask before generating**.
3. Approved segments are **LOCKED**. Never regenerate one to fix a word (regenerating re-rolls other words — e.g. «سامي» became «ساني»). Move older takes to `old/`.
4. **Cheap word test**: generate only the word, or a natural-speed phrase of 3–4 words at most, in **2–3 spelling variants**. Report the cost. The owner picks; add the pick to the lexicon.
5. **Splice only that word** into the approved audio (method below). Every other word stays untouched.
6. If the splice prosody sounds detached, take a slightly longer piece from a **pause-bounded region** of the test instead of the bare word. Never regenerate a long sentence just for prosody: the long context brings the wrong pronunciation back.
7. Always send a **short clip of just the splice point** (about 1.5 s each side) for review, then the whole segment.
8. After approval, rebuild the episode (see Assembly).

### Splice method (ffmpeg + numpy)
Work in WAV at 44.1 kHz mono s16.
```bash
ffmpeg -y -i 08.mp3      -ar 44100 -ac 1 -c:a pcm_s16le seg.wav
ffmpeg -y -i test_b.mp3  -ar 44100 -ac 1 -c:a pcm_s16le word.wav
```
Find word boundaries with energy and zero-crossing rate per 10 ms frame. Cut at a silence or a stop closure (low energy before a burst such as ك/ت/ط/ء). Do not cut inside vowels or fricatives (high ZCR: س/ش/ف/ث).
```python
import numpy as np, subprocess, sys
def load(p):
    raw = subprocess.run(["ffmpeg","-v","quiet","-i",p,"-f","s16le","-ac","1","-ar","44100","-"],
                         capture_output=True).stdout
    return np.frombuffer(raw, np.int16).astype(np.float32) / 32768
x = load(sys.argv[1]); t0, t1 = float(sys.argv[2]), float(sys.argv[3])
F = 441  # 10 ms
for i in range(int(t0*100), int(t1*100)):
    f = x[i*F:(i+1)*F]
    rms = 20*np.log10(np.sqrt((f**2).mean()) + 1e-9)
    zcr = (np.abs(np.diff(np.sign(f))) > 0).mean()
    print(f"{i/100:7.2f}s  {rms:6.1f} dB  zcr {zcr:.2f}  " + "#"*max(0, int(rms+60)//2))
```
Pick `A`,`B` (bad word in seg.wav) and `a`,`b` (good word in word.wav), each at a low-energy frame. Then:
```bash
F=0.005
ffmpeg -y -i seg.wav  -af "atrim=0:A,asetpts=PTS-STARTPTS,afade=t=out:st=$(echo "A-$F"|bc):d=$F" pre.wav
ffmpeg -y -i word.wav -af "atrim=a:b,asetpts=PTS-STARTPTS,afade=t=in:d=$F,afade=t=out:st=$(echo "b-a-$F"|bc):d=$F" mid.wav
ffmpeg -y -i seg.wav  -af "atrim=start=B,asetpts=PTS-STARTPTS,afade=t=in:d=$F" post.wav
ffmpeg -y -i pre.wav -i mid.wav -i post.wav \
  -filter_complex "[0][1][2]concat=n=3:v=0:a=1" -c:a pcm_s16le seg_fixed.wav
```
If the test take is louder or softer, match RMS with `-af volume=XdB`. Do not speed up the inserted word (atempo changes how it sounds).
```bash
ffmpeg -y -ss $(echo "A-1.5"|bc) -t 3.5 -i seg_fixed.wav splice_check.mp3
ffmpeg -y -i seg_fixed.wav -c:a libmp3lame -b:a 192k 08.mp3   # only after approval
```

### Assembly
```bash
ffmpeg -y -f lavfi -i anullsrc=r=44100:cl=mono -t 0.6 -c:a pcm_s16le gap44.wav
for i in 01 02 03 04 05 06 07 08 09 10 11 12; do
  ffmpeg -y -i $i.mp3 -ar 44100 -ac 1 -c:a pcm_s16le w$i.wav; done
( for i in 01 02 03 04 05 06 07 08 09 10 11 12; do echo "file 'w$i.wav'"; \
  [ $i != 12 ] && echo "file 'gap44.wav'"; done ) > list.txt
ffmpeg -y -f concat -safe 0 -i list.txt -c:a libmp3lame -b:a 192k epN.mp3
```
Then update the review sheet's time ranges.

## Never
- Never use IPA in generated text, and never write full tashkeel.
- Never force language `ar`.
- Never regenerate an approved segment, or the whole episode, to fix a word.
- Never change anything the owner did not ask for, including "improving" other words.
- Never convert educated or Standard-Arabic-origin words to hamza (اقتراح، قاموس، مقنع، ثواني…).
- Never write tool or brand names in Arabic letters.
- Never cut a sentence across segments, and never splice in the middle of a voiced sound.
- Never spend credits without stating the cost; when unsure, ask first.
- Never leave a confirmed word out of the lexicon.
- Reply to the owner in Egyptian Arabic.
