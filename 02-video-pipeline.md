# POV Finance Video Pipeline (reusable)

A step-by-step system for making "POV: You…" faceless finance story videos (the Ollie Finance / POV Finance format).
Use it with **Claude + the NexLev MCP connector**, or by hand with the NexLev dashboard, YouTube, and any LLM.

> Background research: see `01-competitor-analysis.md`.
> Voice and data/analytics deep-dives are intentionally out of scope for now. Placeholders are marked ⏸.

---

## 0. Pipeline at a glance

```
[1] Find viral ideas ──► [2] Score & pick ──► [3] Title ──► [4] Script ──► [5] Thumbnail
                                                                   │
[9] Review analytics ◄── [8] Upload ◄── [7] Description/Tags ◄── [6] Scene list + Edit  (⏸ voice)
```

| Step | Output | Time |
|---|---|---|
| 1 Find ideas | 10–20 outlier titles | 15 min |
| 2 Pick | 1 idea + angle | 5 min |
| 3 Title | 1 final + 2 backups | 5 min |
| 4 Script | 2,400–3,300 words | 30–45 min |
| 5 Thumbnail | 1 image + 1 A/B variant | 20 min |
| 6 Edit | 15–21 min video | 2–4 h |
| 7 Metadata | description, tags, hashtags | 5 min |
| 8 Upload | scheduled video | 10 min |
| 9 Review | notes for the next video | 10 min/week |

---

## 1. Find viral ideas (where to scrape titles)

### 1A. Watchlist channels (check every 2–3 days)

| Channel | Channel ID |
|---|---|
| POV Finance | `UCkPZLXcrP3Hc1J-Xd_uN-Mw` |
| Money Life POV | `UCCEwJbv7D1CYzxdKYYOH6AA` |
| Ollie Finance | `UCFGLwmoU1SuT6t_m3gRERpQ` |
| Lucas Grant | `UCTcOboZIhgrKHMDm-oTlTOg` |
| Another Story | `UCr4g4RqDYqMiActaYOhCfCA` |
| J.R. Academy | `UCyW2h6aHuK2fU27JPX3giDA` |
| Finance With Ryan | `UCaPlx4qJW52R7GKQZAMMb8A` |
| Finance With Henry | `UC9wD6-6jjwj2Tmc1oB6oYBA` |
| Finance POV | `UC4zls8vK8iiTSQpnaIxICkg` |
| Liora Invest | `UC0UCPDSC9Rl3y3CSWHmVteA` |
| Quiet Capital | `UCxKdMaBJWSJizkKp27mpQ8g` |
| POV Money Habits | `UCX_LCThQPuuD5a_DiX8PjTw` |

Refresh the watchlist once a month with `get_similar_channels` (see 1D).

### 1B. NexLev calls (in Claude with the NexLev connector)

| Goal | Tool | Parameters |
|---|---|---|
| Outliers on one channel | `youtube_channel_outliers` | `channel_id=<ID>, max_videos=100, min_outlier_threshold=2` |
| Newest uploads on a channel | `youtube_channel_videos` | `channel_id=<ID>, sort_by="newest"` |
| Breakout POV videos YouTube is pushing now | `search_youtube_suggested_videos` | `query="POV", minOutlierScore=2, minPublishDate=<today-30d>, sortBy="outlierScore"` |
| Same, from small channels (easiest to copy) | `search_youtube_suggested_videos` | `query="POV", minOutlierScore=3, maxSubscriberCount=50000` |
| Fresh keyword trends | `youtube_search` | `query="POV you quietly", type="video", upload_date="week", sort_by="views"` |
| Viral small-channel videos | `search_viral_videos_small_channels` | query: "POV finance", "quiet wealth", "old money" |
| Faceless finance outliers | `faceless_outliers_videos` | query: "POV personal finance story" |
| New competitor channels | `get_similar_channels` → poll `get_similar_channels_status` | `channelId=UCkPZLXcrP3Hc1J-Xd_uN-Mw, level=2, async=true` |
| Earnings check | `get_video_rpm` | `videoId=<ID>` |

Rotate these search keywords:
`POV you quietly` · `nobody knows` · `old money` · `retired at` · `rich friend` · `second income` · `richest person at` · `while everyone else` · `paid off house` · `lottery` · `inherited` · `stealth wealth` · `escape rat race`

### 1C. Without Claude (NexLev dashboard / YouTube)
1. NexLev dashboard → outlier/video search: filter to Finance, faceless, published in the last 30 days, outlier ≥ 2x. Search "POV".
2. NexLev → open each watchlist channel → sort by outlier score.
3. YouTube search "POV: You" → Filters → This week → Sort by view count.
4. Save every candidate to a NexLev Swipe File folder called "POV Ideas" (`save_to_swipefile`).

### 1D. Monthly: expand the watchlist
Run `get_similar_channels` on POV Finance and Money Life POV at `level=2`. Add any channel that is **less than 6 months old with more than 500K total views**.

---

## 2. Score & pick the idea

Score each candidate from 0–2 on each line. Make anything that scores **≥ 8 of 12**.

| Criterion | 0 | 1 | 2 |
|---|---|---|---|
| Outlier multiplier | < 2x | 2–4x | > 4x |
| Recency | > 45 days | 15–45 days | < 14 days |
| Copies already made by others | > 10 | 4–10 | < 4 |
| Aspirational (fantasy, not pain) | pain/guilt | mixed | pure fantasy/escape |
| Can add a bigger number / stronger stakes | no | maybe | yes, obviously |
| Fits the "secret / nobody knows" frame | no | partly | yes |

**Angle twist (required).** Never copy the title as-is. Change at least one of these:
- **Number up:** $1M → $10M, retired at 45 → at 35
- **Social stakes:** add a family dinner, a coworker, an ex, or the rich friend
- **Different vehicle:** dividends → rental duplex → small-business acquisition → vending machines
- **Inverted perspective:** "Everyone thinks you're broke…"

---

## 3. Title

### Formula
```
POV: You [Quietly/Silently/Secretly] <aspirational action + 1 specific number> — <Nobody Knows | Everything Changed | Then Things Got Weird | Told No One>
```

### Prompt: title generator
```
You are a YouTube title strategist for a faceless US personal-finance channel that makes
2nd-person "POV: You…" cinematic stories (examples of proven outliers below).

PROVEN OUTLIERS:
- POV: You Adopted the Old Money Mindset — Your Life Changed (556K, 22x)
- POV: You Chose Wealth Over Looking Rich (289K, 11x)
- POV: You Became the Rich Friend — Then Things Got Awkward (356K, 6.6x)
- POV: Your Life After Winning the $600 Million Lottery and You Don't Tell Anyone (170K)
- POV: You Quietly Built a Second Income and Told No One (160K)
- POV: You Pay Off Your House While Everyone Else Still Has a Mortgage (247K)
- POV: You Retired at 40 — Nobody Knows (111K)

SOURCE IDEA I'M ADAPTING: "<paste competitor outlier title + views>"

Write 15 new titles that keep the same viewer desire but escalate ONE thing
(bigger number, higher social stakes, or a fresher wealth vehicle). Rules:
- Start with "POV: You" (or "POV: Your")
- 45–85 characters, Title Case, use an em dash "—" before the payoff phrase
- Exactly one specific number (age, $ amount, or years)
- No clickbait lies; the story must be able to deliver it
- Perfect grammar
Then rank your top 3 by predicted CTR and explain each in one line.
```

Pick #1, and keep #2 and #3 for YouTube's thumbnail/title A/B test.

---

## 4. Script

### 4A. Spec (the house style)
- **2,400–3,300 words** (15–21 min at ~160 wpm)
- 2nd person, present tense, cinematic and sensory. **No** listicles, headings, or "tip #1".
- Teach 3–6 real finance concepts *through the story's action*
- Time-jump markers every 300–450 words ("Three weeks in…", "Six months later…")
- Specific numbers everywhere, and they must be internally consistent (the math has to add up)
- One near-exposure / conflict beat around 50–60% of the way through
- Emotional cost (loneliness, double life) before the quiet win
- Ending: moral → like/subscribe → **one specific comment question**
- Short paragraphs. Use sentence fragments for punch, but don't overuse "Not X. Not Y. Just Z." (cap it at 3 per script)

### 4B. Beat sheet (fill this in first)

| % | Beat | Your notes |
|---|---|---|
| 0–3% | Cold open: exact time + mundane place + small humiliation | |
| 3–8% | The trigger: wealth arrives or the plan starts | |
| 8–10% | Secrecy rule stated (the title promise, paid off) | |
| 10–35% | Mechanics: finance concept #1–3 shown in action, with numbers | |
| 35–50% | Time jumps, the double life, the first visible result | |
| 50–60% | Near-exposure / conflict / external shock (layoffs, nosy friend) | |
| 60–80% | Emotional cost plus finance concept #4–6 | |
| 80–95% | Quiet victory (no flex), helping someone anonymously | |
| 95–100% | Moral line → CTA → comment question | |

### 4C. Prompt: outline
```
Act as a head writer for a faceless YouTube channel that tells 2nd-person, present-tense
cinematic finance stories ("POV: You…"). Audience: US adults 25–45 with salaried jobs.

TITLE: <final title>
FINANCE CONCEPTS TO TEACH (pick 3–6): <e.g. revocable trust, lump sum vs annuity, index funds,
municipal bonds, savings rate, lifestyle inflation>

Produce a beat-by-beat outline (12–16 beats) following this structure:
cold open (exact time + mundane place + small humiliation) → trigger → secrecy rule →
mechanics with real numbers → time jumps (1 week, 3 weeks, 4 months, 6 months, 18 months...) →
near-exposure conflict at ~55% → emotional cost → quiet victory without flexing →
moral + like/subscribe + one specific comment question.

For each beat give: timestamp %, setting, what happens, the exact numbers used, which
finance concept is taught. Make sure all money math is internally consistent and realistic
for the US (taxes, returns 4–8%/yr, etc.). Invent a protagonist age and job, and 1–2 side
characters with first names.
```

### 4D. Prompt: full script
```
Write the full narration script from the outline above.

STYLE RULES
- 2nd person, present tense, cinematic, sensory details (sounds, light, smells, objects).
- 2,800–3,200 words. Plain narration only: no headings, no stage directions, no emojis.
- Short paragraphs (1–4 sentences). Vary rhythm: long sentence, then a short punch.
- Every 300–450 words, open a paragraph with a time marker ("Three weeks in, ...").
- Teach each finance concept through action, then in ONE plain sentence explain why it matters.
- Use specific numbers (dollars, days, ages, counts). Keep the math consistent.
- Max 3 uses of the "Not X. Not Y. Just Z." pattern. Avoid clichés: "game-changer",
  "journey", "unlock", "level up", "in today's world".
- First 2 sentences must place the viewer in a specific time + place + small problem.
- Within the first 60 seconds, state the secret ("You don't tell anyone.").
- Ending: one quotable moral line → "If this story made you think differently about
  <topic>, hit like and subscribe for more stories like this." → ONE specific,
  personal comment question tied to the story's dilemma.
- Educational framing only; no promises of returns; no specific stock picks.
```

### 4E. Script QA checklist (before recording)
- [ ] Hook names a time, a place, and a problem in the first 2 lines
- [ ] Title promise delivered in the first 60 s
- [ ] ≥ 3 real finance concepts, explained correctly
- [ ] All numbers add up (check with a calculator: compounding, taxes, timelines)
- [ ] ≥ 5 time-jump markers
- [ ] Conflict beat present at ~50–60%
- [ ] No more than 3 "Not X… Just Z" lines
- [ ] Ends with a specific comment question
- [ ] Word count 2,400–3,300
- [ ] Read 3 random paragraphs aloud and confirm they sound human, not robotic
- [ ] Original: different characters, numbers, and setting from the source video

---

## 5. Thumbnail

### Style rules (niche standard)
- 2D cartoon / flat illustration. One faceless or simple-faced character, calm or smug expression.
- A single symbolic scene that tells the whole title (desk + hidden money, house + "PAID OFF", lottery ticket glowing)
- High contrast. Dark/muted background with **one** bright accent (gold/green money, warm window light)
- **0–4 words** of text, or one big number ("$600M", "AGE 40"), bold white with a dark outline, top-left or right third
- Readable at mobile size (168×94). Test it by zooming out.
- Don't repeat the title words in the thumbnail. It should *complement* the title.

### Prompt: thumbnail concepts
```
Title: <final title>
Give me 5 YouTube thumbnail concepts in 2D flat cartoon illustration style for a faceless
finance POV channel. For each: (1) scene in one sentence, (2) character pose/expression,
(3) color palette (dark background + one accent), (4) 0–4 words of overlay text that
COMPLEMENT (not repeat) the title, (5) why it creates curiosity. Then write a ready-to-use
image-generation prompt (16:9, 1280x720) for the best 2 concepts.
```

Example image prompt:
```
2D flat vector cartoon illustration, 16:9. A calm young man in a plain gray hoodie sits in a
small dim office cubicle, looking slightly to the side with a subtle knowing smile. Under his
desk, hidden from coworkers, a glowing open briefcase full of gold coins and cash lights his
face from below. Coworkers in the background are blurred and unaware. Dark navy and charcoal
palette, strong warm gold accent light, clean bold outlines, minimal detail, high contrast,
YouTube thumbnail composition with empty space top-right for text. No text in the image.
```
Add the text afterwards in Canva or Photoshop. Tools: Midjourney / ChatGPT image / Ideogram / Leonardo, or NexLev `generate_thumbnail` → `get_thumbnail_generation_status`. Use NexLev `get_similar_thumbnails` (queryType="text") to see what already exists for your concept.

### Thumbnail QA
- [ ] Story is understandable in < 1 second at mobile size
- [ ] One focal point, one accent color
- [ ] ≤ 4 words
- [ ] Clearly different from the source video's thumbnail
- [ ] A/B variant made (YouTube "Test & Compare")

---

## 6. Scene list & editing

### 6A. Prompt: script → scene list
```
Split the script below into scenes for a 2D-illustrated faceless YouTube video.
One scene = one sentence or clause, ~4–8 seconds of narration (≈10–20 words).
Output a table: scene #, narration text, image prompt, motion, on-screen text, SFX.
- Image prompt: 2D flat cartoon illustration, consistent main character
  "<describe once: age, hair, clothes>", consistent palette "<palette>", 16:9.
- Motion: one of [slow zoom in, slow zoom out, pan left, pan right, static + character anim].
- On-screen text: ONLY for money amounts, ages, and time jumps (e.g. "$40,000 / DAY",
  "6 MONTHS LATER"). Otherwise leave blank.
- SFX: only at reveals, money moments, or time jumps (whoosh, soft riser, cash register, phone buzz).
SCRIPT: <paste>
```
Expect about 150–250 scenes for a 17-minute video.

### 6B. Editing spec (house style)
*Inferred from the niche. Confirm with the 6C checklist and adjust.*

| Element | Spec |
|---|---|
| Visual medium | 2D flat cartoon/illustrated stills, one recurring faceless character |
| Visual change | Every 4–8 s; faster (2–3 s) in the first 30 s and at tense moments |
| Motion | Ken Burns zoom/pan on every still; alternate directions; 5–10% scale |
| Transitions | Mostly hard cuts; a soft cross-dissolve for time jumps |
| Text on screen | Big bold numbers when money is spoken; "X MONTHS LATER" cards; no full subtitles burned in |
| Music | Soft piano / ambient lo-fi bed at −24 to −28 dB under the voice; swap tracks at the conflict beat and the victory beat |
| SFX | Whoosh on time jumps, a riser before reveals, phone buzz, cash/coin sounds on money moments; keep them subtle |
| Intro | None. Start on the cold open at frame 1. |
| Outro | Last 20 s: end screen with 2 video elements + subscribe |
| Color/overlay | Optional light film grain and vignette for the "cinematic" feel |
| Export | 1080p or 4K, 24/30 fps, −14 LUFS loudness |

Tools: CapCut / Premiere / DaVinci Resolve. Image generation: Midjourney / Leonardo / ChatGPT image (keep a fixed "character sheet" prompt so the protagonist stays consistent).

### 6C. Editing-style verification checklist (do once per competitor)
Watch the first 2:30 of a competitor outlier (e.g. Ollie `lzA2Ie-vxKU`, POV Finance `avWK82b0fUU`) and fill this in:

| Question | Ollie | POV Finance |
|---|---|---|
| Medium (2D / AI-realistic / stock / mix) | | |
| Seconds per image (count images in 60 s ÷ 60) | | |
| Motion on stills | | |
| Character animation? | | |
| Text on screen: when/style | | |
| Subtitles burned in? | | |
| Music genre | | |
| SFX used | | |
| Grain / vignette / grading | | |

Then update 6B. In Claude with NexLev you can also run `watch_youtube_video_and_ask` with `startOffset="0s", endOffset="150s"` and ask for exactly these points (it has a daily limit).

### 6D. Voice
See `03-production-bible.md` §2. Target 155–162 wpm (measured on Ollie's winners), a calm documentary narrator, ElevenLabs settings and TTS script markup.

---

## 7. Description, tags, hashtags

### Prompt
```
Write YouTube metadata for this video.
Title: <title>
Script summary: <5 bullets>
Finance concepts covered: <list>

1) DESCRIPTION (180–260 words):
   - Line 1–2: a curiosity hook question or "What if…" (this shows in search)
   - Para: "In this video, we break down…" naming the real concepts covered
   - "You will learn:" + 4–5 bullets (middle dot •)
   - Separator line
   - ⚠️ Disclaimer: "This content is intended for educational purposes only and should
     not be considered financial advice. Always consult a licensed financial advisor
     before making investment decisions."
   - "Subscribe for more POV stories about money, wealth, and the quiet decisions that change your life."
   - 5–8 hashtags: 3 topic + #PersonalFinance + #povfinance #povstorytelling
2) TAGS: 10–14 comma-separated: 6 search-intent phrases people actually type
   (e.g. "how to build wealth", "what happens if you win the lottery") + niche identity tags
   ("POV story telling", "POV finance usa", "usa finance", "personal finance explained").
3) PINNED COMMENT: the story's comment question + one follow-up question.
```

Later upgrade (once you reach 1K+ subs): put a lead magnet at the very top of the description (free calculator / Notion budget template / newsletter), the way POV Finance does.

---

## 8. Upload checklist
- [ ] Title (final) + 2 alternates loaded in "Test & Compare"
- [ ] Thumbnail A/B (up to 3)
- [ ] Description + tags + hashtags
- [ ] Category: **Education** · Language: English · Audience: not made for kids
- [ ] "Altered or synthetic content" disclosure set to **Yes** if AI voice or realistic AI visuals are used
- [ ] Chapters: optional (competitors don't use them; test it)
- [ ] End screen + subscribe element
- [ ] Pinned comment ready
- [ ] Schedule for **US prime time**: 12:00–15:00 ET (9:00 PM–12:00 AM PKT)
- [ ] Log the video in your tracker (step 9)

Cadence target: **3–4 videos/week**. This is an outlier game, so volume compounds.

---

## 9. Review loop (weekly)

Track every video in a sheet:

| Date | Title | Source idea + its multiplier | Angle twist | Length | 48h views | 7d views | CTR | AVD % | Outlier vs your avg |
|---|---|---|---|---|---|---|---|---|---|

Every week:
1. In NexLev, `list_my_youtube_channels` → `get_my_top_videos` / `get_my_video_analytics` / `get_my_audience_retention` for your new uploads
2. **CTR < 4%** → swap the thumbnail first, then the title
3. **Retention drop in the first 30 s** → rewrite the cold-open formula
4. **Retention drop mid-video** → add a conflict beat earlier or tighten the time jumps
5. When a video hits **> 2x your average**, make 2 follow-ups on the same angle within 7 days (number up, or a different social stake)
6. Add your own winners and losers to the "title bank" in `01-competitor-analysis.md`

---

## 10. One-shot master prompt (Claude + NexLev)

Paste this into Claude with the NexLev connector enabled to run steps 1–3 automatically:

```
You are my YouTube research assistant for a faceless "POV: You…" US personal-finance story channel.

1. For each of these channels, run youtube_channel_outliers (max_videos=60,
   min_outlier_threshold=2) and youtube_channel_videos (sort_by=newest):
   UCkPZLXcrP3Hc1J-Xd_uN-Mw, UCCEwJbv7D1CYzxdKYYOH6AA, UCFGLwmoU1SuT6t_m3gRERpQ,
   UCTcOboZIhgrKHMDm-oTlTOg, UCr4g4RqDYqMiActaYOhCfCA, UCyW2h6aHuK2fU27JPX3giDA,
   UC9wD6-6jjwj2Tmc1oB6oYBA, UCxKdMaBJWSJizkKp27mpQ8g
2. Run search_youtube_suggested_videos with query="POV", minOutlierScore=2,
   minPublishDate=<30 days ago>, sortBy="outlierScore", limit=50.
3. Run youtube_search query="POV you quietly" upload_date="week" sort_by="views".
4. Deduplicate. Keep videos published in the last 45 days with ≥2x outlier score.
5. Score each with this rubric (0–2 each): multiplier, recency, # of existing copies
   (check with youtube_search on the exact title), aspirational vs pain,
   room to escalate the number/stakes, fits "secret / nobody knows". Show a table.
6. For the top 3 ideas, propose 5 escalated titles each, following:
   "POV: You [Quietly/Silently] <action + 1 number> — <payoff tag>", 45–85 chars.
7. Recommend ONE to make this week and give a 12-beat outline for it.
```

Then continue manually with steps 4D (full script) → 5 → 6 → 7 → 8.

---

## 11. Rules that keep the channel safe
- **Original stories only.** Never re-voice, translate, or paraphrase a competitor's script. Use the *title idea* as inspiration, then write new characters, numbers, and settings.
- Make each video meaningfully different (YouTube's "inauthentic content" policy targets mass-produced, repetitive templates).
- Keep the educational framing and the disclaimer. No specific stock/crypto picks or guaranteed returns.
- Disclose synthetic content where YouTube requires it.
