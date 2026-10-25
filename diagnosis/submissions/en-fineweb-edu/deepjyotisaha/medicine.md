# Medicine sheet: en-fineweb-edu (deepjyotisaha)

Written from the first 150 labelled documents of the pack (500 in all). Every number below is produced by `medicine.py`, which runs each rule against the labels in `labels.csv` and against the whole pack. Diseases found more than five times so far: **C1** and **C3**.

## Medicine 1: C1 boilerplate

1. **Disease:** C1, in 79 of the 150 labelled documents (87 labelled cases). Three examples:
   - `74ea30694d60f739` (document 13): "© 1995-2017 Bible History Online"
   - `f6393483c601faf0` (document 22): "View all Kids Bikes"
   - `1e5e295e77f6c38e` (document 45): "By Kevin Schargen"

2. **Medicine:** Remove a line, or the end of a line, when it is website text rather than article text. Five kinds, each with its own pattern (all case-insensitive, lines under 120 characters unless stated):
   - teaser and action links: a line that starts with "Read more", "Continue reading", "Explore further", "Click here", "Learn more about", "To learn more about", "Find out more", "For further information", "View all", "View larger map", "Show summary details", "Share this", "Choose a citation style" or "Call now"; and a teaser such as "[...] Read more", "Continue reading →" or "Read more »" at the end of a longer line (only the teaser is removed);
   - bylines and dates: in the first three lines, a line that is "By <Name>", "From: …", "Posted …", "Updated …", "Published …", "Contributed by …", "Written by …", "Latest revision …" or only a date; anywhere, a line that is only "By <Name>";
   - credits, sources and footers: a line that starts with "©", "Copyright", "Source:", "Article source:", "Website:", "Courtesy:", "Photo:", "Image:", "Editor's note:", "Editor:" or "Correction";
   - navigation: a breadcrumb or menu line of four or more items joined by " | ", " : " or " » " (under 160 characters), and link boxes such as "|… »||… »|";
   - shop and promotion text (under 220 characters): book details ("Paperback:", "Publisher:", "ISBN-13:", "Product dimensions:"), order confirmations, "Help support … and get …", "Don't forget to check out …", "click on the … to follow me", "Subscribe to our newsletter".
   - line scorer: every line also gets a score from 0 to 1 from a logistic regression trained on the labels (positive: the labelled C1 lines; negative: every line of the documents labelled clean). Its features are what boilerplate lines share whatever their words: length, a full stop at the end, position (first or last three lines), link words, web address, ©, date, separators, share of capitalised words, digits, whether a paragraph follows, and how many other documents repeat the line. A line scoring 0.97 or more is removed, but only if it is at most 200 characters and carries an explicit boilerplate signal (link word, web address, ©, date, byline or separator), so an article title or an ordinary sentence is never removed on its score alone;
   - block expansion: a line next to a removed line is removed too if it is short, has no full stop, scores at least half the threshold, and is signalled or a "Key: value" line (the rest of a byline or of a book-details block);
   - a teaser glued to the end of a sentence ("…can be prescribed.Learn more about …") removes only the teaser, never the sentence;
   - guard: if all of this would remove more than 20% of a document and more than 300 characters, nothing is cut and the document is flagged for drop (so a normal footer on a short page is cut, not turned into a drop). The threshold 0.97 is the one that catches the most labelled lines while changing no clean document, in a five-fold test on documents the scorer did not see during training (`c1tune.py`).

   In code: the function `c1_spans(text)` in **Runnable code** at the end of this sheet.

3. **Action:** fix. The matched text is removed; the rest of the document is kept.

4. **Harm test:** proxy gold set = the 59 rows of `labels.csv` with verdict `clean` (each read in full, no disease found). The medicine changes or drops **0 of 59** of them (0.0%, each tested by a scorer that had not seen it): **pass** against the 1% bar. On the whole pack it changes 150 of 500 documents and removes 13,477 of 2,442,877 characters (0.55%). The official gold set for English (owned by Data A) is not published yet; this test will be repeated on it when it is.

5. **Side effects:** on documents the scorer had not seen, the medicine finds 53 of the 86 labelled C1 lines (62%; the phrase rules alone find 47). It misses C1 that has no fixed wording and no boilerplate signal: one-off site headings ("Programs for children", "Chapter 02 Worksheet"), reference lists and citations, catalogue records and promotion inside a long paragraph. These are left to the readers. Article titles are kept on purpose: the first version of the scorer removed titles and sentences such as "By the end of 1992, …" from clean documents, which is why the scorer may only remove signalled lines. Remaining risks: a "Source:" line under a chart, and a short line with a date or a link word that belongs to the article.

## Medicine 2: C3 truncated document

1. **Disease:** C3, in 6 of the 150 labelled documents (6 labelled cases). Three examples:
   - `2067313a5e000336` (document 5): "He writes in the Guardian on Tuesday:"
   - `8620d74ba9ae6cdc` (document 24): "For more information about the collection, visit"
   - `d9cf118c7cae38e0` (document 42): "week we celebrate the holiday of Chanuka."

2. **Medicine:** Remove the last line of a document when it is an unfinished sentence: the last non-empty line has at least six words and ends on a word no complete sentence ends on (for example "the", "of", "to", "visit", "they") or on ":" or ",", and the C1 medicine does not already remove it. The document then ends at its last complete line. Run after the C1 medicine. Two broader rules were tested and rejected (25 Oct): "the last line ends with any lower-case letter" also removed titles and author notes at the end of good documents, and "the text starts with a lower-case letter" also removed dictionary entries that start in lower case. Those cases stay with the readers.

   In code: the function `c3_spans(text)` in **Runnable code** at the end of this sheet.

3. **Action:** fix. The matched text is removed; the rest of the document is kept.

4. **Harm test:** proxy gold set = the 59 rows of `labels.csv` with verdict `clean` (each read in full, no disease found). The medicine changes or drops **0 of 59** of them (0.0%): **pass** against the 1% bar. On the whole pack it changes 3 of 500 documents and removes 268 of 2,442,877 characters (0.01%). The official gold set for English (owned by Data A) is not published yet; this test will be repeated on it when it is.

5. **Side effects:** the medicine finds 1 of the 6 labelled cases. It misses cut-offs that do not end on a dangling word or a colon (for example a teaser ending in "..."), and documents that start in the middle of a sentence. Some labelled cut-offs are removed by the C1 medicine first, because their last line is also a link ("…For more information about the collection, visit"). On the whole pack it changes 3 documents; checked on 25 Oct, one of them is a title ending in "On" ("The Saga Lives On"), which is a wrong removal.

## Runnable code

Both medicines in one self-contained Python file: copy it into `medicine_standalone.py` and it runs with the standard library alone (no numpy, no other files). It is generated from the code and the trained weights that were tested above, and it gives the same result as that code on all 500 documents of the pack (0 differences, checked by `standalone.py`).

How to use it:

```python
from medicine_standalone import clean, c1_spans, c3_spans

cleaned, action = clean(text)        # action: "keep", "fix" or "drop"
spans, flagged = c1_spans(text)      # what C1 removes: (start, end, rule); flagged = drop
c3 = c3_spans(text)                  # what C3 removes
```

To score a single line: `lines, X = line_features(text)` gives every line with its 20 features, and `line_score(x)` gives its score from 0 to 1. A line is removed by the scorer only if the score is at least `THRESHOLD`, the line has at most 200 characters and `gate_signal(line)` is true.

```python
"""C1 (boilerplate) and C3 (truncated document) medicines for en-fineweb-edu. Self-contained: needs only the
Python standard library. Usage: cleaned_text, action = clean(text)
"""
import math
import re

# ----- C1 phrase rules -----
SHORT = 120
START_LINES = 3
C1_RULES = {
    'C1-teaser-line': (re.compile('(?i)^\\W{0,6}(?:read more|continue reading|explore further|click here|learn more about|to learn more about|find out more|for further information|view all\\b|view larger map|show summary details|share this|choose a citation style|call now)'), 'anywhere'),
    'C1-teaser-inline': (re.compile('(?i)(?:\\[\\.\\.\\.\\]\\s*read more\\.?|\\s+continue reading\\s*→?|\\s*\\bread more\\s*»?|(?<=[.!?])\\s*learn more about [^.!?]{0,60})\\s*$'), 'inline'),
    'C1-byline-alone': (re.compile("^[Bb]y\\s+[A-Z][\\w.'-]+(?:\\s+[A-Z][\\w.'-]+){0,3}$"), 'anywhere'),
    'C1-byline': (re.compile("(?i)^(?:by\\s+[A-Z][\\w.'-]+(?:\\s+[A-Z][\\w.'-]+){0,3}\\b.*|from:\\s.*|posted:?\\s.*|updated\\s.*|published\\s.*|contributed by\\s.*|written by\\s.*|latest revision\\s.*|(?:(?:Monday|Tuesday|Wednesday|Thursday|Friday|Saturday|Sunday),?\\s+)?(?:(?:January|February|March|April|May|June|July|August|September|October|November|December)\\s+\\d{1,2},?\\s+\\d{4}|\\d{1,2}\\s+(?:January|February|March|April|May|June|July|August|September|October|November|December)\\s+\\d{4})\\.?)$"), 'start'),
    'C1-credit': (re.compile("(?i)^\\W{0,3}(?:©|copyright\\b|\\(c\\)\\s*\\d{4}|source:|article source:|website:|courtesy:|photo:|image:|editor'?s note:|editor:|correction|published\\s+\\d{1,2}\\s)"), 'anywhere'),
    'C1-navigation': (re.compile('^(?=.{0,160}$)(?:[^|:»\\n]+\\s(?:\\||:|»)\\s){3,}[^|:»\\n]+$|^.*»\\s*\\|\\|.*»\\s*\\|?$'), 'anywhere'),
    'C1-promotion': (re.compile("(?i)^\\W{0,3}(?:paperback:|hardcover:|publisher:|language:\\s*\\w+$|isbn-1[03]:|product dimensions:|indian edition also available|we've sent an email with your order|help support .{0,40} and get|don't forget to check out|click on the .{0,30} to follow me|subscribe to (?:our|the) newsletter)"), 'anywhere'),
}

# ----- C1 line scorer: feature patterns -----
LINK_WORDS = re.compile('(?i)\\b(?:read more|click|view (?:all|full|larger|more|current|details|profile|cart)|share|follow|subscribe|sign up|log ?in|register|download|visit|learn more|find out|contact|call now|order|buy|shop|newsletter|comment|rate|advertis|explore further|continue reading|related|see also|tags?)\\b')
URL = re.compile('(?i)https?://|www\\.|\\.(?:com|org|net|edu|gov|co\\.uk)\\b')
COPY = re.compile('(?i)©|\\bcopyright\\b|all rights reserved')
DATE = re.compile('(?i)\\b(?:jan|feb|mar|apr|may|jun|jul|aug|sep|oct|nov|dec)[a-z]*\\.?\\s+\\d{1,2},?\\s+\\d{4}\\b|\\b\\d{1,2}\\s+(?:jan|feb|mar|apr|may|jun|jul|aug|sep|oct|nov|dec)[a-z]*\\s+\\d{4}\\b|\\b(?:19|20)\\d{2}\\b')
SEPARATORS = re.compile('\\s\\|\\s|\\|\\||»|\\s:\\s|\\s/\\s')
BYLINE = re.compile("^(?:[Bb]y\\s+[A-Z][\\w.'-]+(?:\\s+[A-Z][\\w.'-]+)*(?:,|$)|(?i:from|posted|updated|published|written by|contributed by|photo|image|source|courtesy)\\s*:|(?i:posted|updated|published)\\s+(?:on\\s+)?\\d)")
KEY_VALUE = re.compile('^[-•*]?\\s*[A-Z][\\w -]{1,25}:\\s+\\S')
FULL_DATE = re.compile('(?i)\\b(?:jan|feb|mar|apr|may|jun|jul|aug|sep|oct|nov|dec)[a-z]*\\.?\\s+\\d{1,2},?\\s+\\d{4}\\b|\\b\\d{1,2}\\s+(?:jan|feb|mar|apr|may|jun|jul|aug|sep|oct|nov|dec)[a-z]*\\s+\\d{4}\\b|\\b\\d{1,2}/\\d{1,2}/\\d{2,4}\\b')
MENU_SEP = re.compile('[A-Za-z]\\s(?:\\||»|:)\\s[A-Za-z]|»')
END_PUNCT = '!").?’”'

# ----- C1 line scorer: trained weights (c1tune.py, 25 Oct) -----
THRESHOLD = 0.97
B = -3.0300901064718957
# (weight, mean, standard deviation) for each feature, in the order of FEATURES
FEATURES = ['short_60', 'short_120', 'log_length', 'ends_sentence', 'first_3', 'last_3', 'rel_position', 'link_words', 'url', 'copyright', 'date', 'separators', 'capitalised', 'bullet', 'byline_start', 'repeated_docs', 'digits', 'few_words', 'next_is_paragraph', 'prev_is_paragraph']
WEIGHTS = [
    (-0.30069947477267067, 0.2542372881355932, 0.43543161383610707),  # short_60
    (0.3705660753723984, 0.41460234680573665, 0.4926532673333473),  # short_120
    (-1.3633854117995294, 4.973915804006418, 1.1158612815504103),  # log_length
    (-0.4767146479133662, 0.7340286831812256, 0.44184904247057094),  # ends_sentence
    (1.1036742053572959, 0.26988265971316816, 0.4438986490034759),  # first_3
    (1.7910747321417884, 0.288135593220339, 0.4528945506911009),  # last_3
    (-0.3419316526454081, 0.5102084982135137, 0.336970341150049),  # rel_position
    (1.0158531038667535, 0.09908735332464146, 0.347217213837886),  # link_words
    (0.0984650911683374, 0.00651890482398957, 0.0804761384811526),  # url
    (0.08784538269706771, 0.01303780964797914, 0.1134364376839954),  # copyright
    (-0.250628100332234, 0.0938722294654498, 0.2916508779070208),  # date
    (0.7449039958376682, 0.04302477183833116, 0.42963885745314495),  # separators
    (1.0268636332746048, 0.19286363562093278, 0.22020155725876003),  # capitalised
    (-0.28936205508947666, 0.20208604954367665, 0.4015560719583507),  # bullet
    (1.0346790652206754, 0.01955671447196871, 0.138471114922833),  # byline_start
    (0.0, 0.0, 1e-09),  # repeated_docs
    (1.154775046239996, 0.010813355752272085, 0.034804250997227164),  # digits
    (-0.38352276417640513, 0.06779661016949153, 0.2513965598845974),  # few_words
    (-1.1926348206232442, 0.4198174706649283, 0.49352888768185316),  # next_is_paragraph
    (0.04584941941124311, 0.4380704041720991, 0.4961499029052537),  # prev_is_paragraph
]
MAX_LINE = 200
MAX_SHARE = 0.2
MIN_DROP_CHARS = 300
EXPAND_FACTOR = 0.5

# ----- C3 -----
DANGLING = ['a', 'about', 'an', 'and', 'are', 'as', 'at', 'be', 'because', 'been', 'but', 'by', 'can', 'could', 'for', 'from', 'had', 'has', 'have', 'he', 'her', 'his', 'i', 'if', 'in', 'including', 'into', 'is', 'it', 'its', 'more', 'most', 'not', 'of', 'on', 'or', 'our', 'see', 'she', 'such', 'than', 'that', 'the', 'their', 'these', 'they', 'this', 'those', 'to', 'very', 'visit', 'was', 'we', 'were', 'when', 'which', 'while', 'who', 'whose', 'will', 'with', 'would', 'you', 'your']

# ----- helpers -----------------------------------------------------------------------------------------

def _raw_lines(text):
    """(offset, line) for every line, empty ones included."""
    pos = 0
    for line in text.split("\n"):
        yield pos, line
        pos += len(line) + 1


def _lines(text):
    """(offset of the stripped line, stripped line) for every non-empty line."""
    out = []
    for pos, raw in _raw_lines(text):
        s = raw.strip()
        if s:
            out.append((pos + raw.index(s), s))
    return out


def _norm(line):
    return " ".join(line.lower().split())


# ----- C1 part 1: the phrase rules -----------------------------------------------------------------------

def c1_phrase_spans(text):
    """(start, end, rule) of every span the phrase rules remove."""
    spans, k = [], 0
    for off, line in _raw_lines(text):
        stripped = line.strip()
        if not stripped:
            continue
        k += 1
        for rule, (rx, where) in C1_RULES.items():
            if where == "inline":
                m = rx.search(line)
                if m and len(stripped) > SHORT:
                    spans.append((off + m.start(), off + m.end(), rule))
                    break
                continue
            if where == "start" and k > START_LINES:
                continue
            limit = {"C1-navigation": 160, "C1-promotion": 220}.get(rule, SHORT)
            if len(stripped) > limit or not rx.search(stripped):
                continue
            spans.append((off, off + len(line), rule))
            break
    return spans


# ----- C1 part 2: the line scorer --------------------------------------------------------------------------

def line_features(text, repetition=None):
    """The 20 features of every non-empty line, in the order of FEATURES.
    repetition: optional {normalised line: number of documents containing it}; without it every line counts once."""
    repetition = repetition or {}
    lines = _lines(text)
    n = len(lines)
    feats = []
    for k, (off, s) in enumerate(lines):
        words = s.split()
        caps = sum(1 for w in words if w[:1].isupper()) / max(1, len(words))
        nxt = len(lines[k + 1][1]) if k + 1 < n else 0
        prv = len(lines[k - 1][1]) if k > 0 else 0
        feats.append([
            float(len(s) <= 60), float(len(s) <= 120), math.log(1 + len(s)), float(s[-1] in END_PUNCT),
            float(k < 3), float(k >= n - 3), k / max(1, n - 1),
            float(len(LINK_WORDS.findall(s))), float(bool(URL.search(s))), float(bool(COPY.search(s))),
            float(bool(DATE.search(s))), float(len(SEPARATORS.findall(s))), caps,
            float(s[:1] in "-•*|►>"), float(bool(BYLINE.match(s))),
            math.log(max(1, repetition.get(_norm(s), 1))), sum(ch.isdigit() for ch in s) / len(s),
            float(len(words) <= 3), float(nxt > 200), float(prv > 200),
        ])
    return lines, feats


def line_score(x):
    """The probability that a line is boilerplate: logistic regression on standardised features."""
    z = B + sum(w * (v - m) / s for v, (w, m, s) in zip(x, WEIGHTS))
    return 1 / (1 + math.exp(-z))


def gate_signal(line):
    """An explicit boilerplate signal: link words, web address, ©, a full date, a byline or menu separators.
    A table row "|cell||cell|" without link words is content."""
    if line.startswith("|") and "||" in line and "»" not in line and not LINK_WORDS.search(line):
        return False
    return bool(LINK_WORDS.search(line) or URL.search(line) or COPY.search(line) or FULL_DATE.search(line)
                or BYLINE.match(line) or MENU_SEP.search(line))


def c1_spans(text, repetition=None):
    """The C1 medicine. Returns (spans to remove, flagged_for_drop)."""
    phrase = c1_phrase_spans(text)
    phrase_hits = {s for s, _, _ in phrase}
    lines, X = line_features(text, repetition)
    scores = [line_score(x) for x in X]
    inline = {k for k, (o, s) in enumerate(lines)
              if any(o <= a < o + len(s) for a, _, r in phrase if r == "C1-teaser-inline")}
    remove = {k for k, (off, s) in enumerate(lines) if off in phrase_hits}
    remove |= {k for k, sc in enumerate(scores)
               if sc >= THRESHOLD and len(lines[k][1]) <= MAX_LINE and gate_signal(lines[k][1]) and k not in inline}
    changed = True
    while changed:                          # block expansion
        changed = False
        for k in sorted(remove):
            for j in (k - 1, k + 1):
                if 0 <= j < len(lines) and j not in remove and j not in inline and len(lines[j][1]) <= 120 \
                        and lines[j][1][-1] not in END_PUNCT and scores[j] >= EXPAND_FACTOR * THRESHOLD \
                        and (gate_signal(lines[j][1]) or KEY_VALUE.match(lines[j][1])):
                    remove.add(j)
                    changed = True
    spans = [(lines[k][0], lines[k][0] + len(lines[k][1]), "C1-phrase" if lines[k][0] in phrase_hits else
              "C1-scorer") for k in sorted(remove)]
    spans += [sp for sp in phrase if sp[2] == "C1-teaser-inline"]
    removed = len({p for s, e, _ in spans for p in range(s, e)})
    if text and removed > MAX_SHARE * len(text) and removed > MIN_DROP_CHARS:
        return [], True                      # too much would go: drop the document instead
    return spans, False


# ----- C3 ------------------------------------------------------------------------------------------------

def c3_spans(text, repetition=None):
    """The C3 medicine: remove the last line when it is an unfinished sentence and C1 does not remove it."""
    lines = [(o, l) for o, l in _raw_lines(text) if l.strip()]
    if not lines:
        return []
    off, last = lines[-1]
    c1, flagged = c1_spans(text, repetition)
    if flagged:
        c1 = [(0, len(text), "C1-drop-flag")]
    if any(s < off + len(last) and e > off for s, e, _ in c1):
        return []
    s = last.rstrip()
    words = s.split()
    if len(words) >= 6 and (s[-1] in ":," or words[-1].lower() in DANGLING):
        return [(off, off + len(last), "C3-unfinished-end")]
    return []


# ----- use ---------------------------------------------------------------------------------------------------

def clean(text, repetition=None):
    """Apply both medicines. Returns (cleaned text, action) where action is "keep", "fix" or "drop"."""
    spans, flagged = c1_spans(text, repetition)
    if flagged:
        return "", "drop"
    spans = spans + c3_spans(text, repetition)
    if not spans:
        return text, "keep"
    for s, e in sorted({(s, e) for s, e, _ in spans}, reverse=True):
        text = text[:s] + text[e:]
    return text, "fix"


if __name__ == "__main__":
    sample = ("By Kevin Schargen\nThe article starts here and runs on for a while, as articles do. It has several "
              "sentences of real content.\n© 2015 Example Publishing\nView all Kids Bikes")
    print(clean(sample))
```
