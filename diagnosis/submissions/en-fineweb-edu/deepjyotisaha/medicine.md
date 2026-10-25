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

   In code (Python):

```python
import re

C1_PATTERNS = {
    'C1-teaser-line': ('(?i)^\\W{0,6}(?:read more|continue reading|explore further|click here|learn more about|to learn more about|find out more|for further information|view all\\b|view larger map|show summary details|share this|choose a citation style|call now)', 'anywhere'),
    'C1-teaser-inline': ('(?i)(?:\\[\\.\\.\\.\\]\\s*read more\\.?|\\s+continue reading\\s*→?|\\s*\\bread more\\s*»?|(?<=[.!?])\\s*learn more about [^.!?]{0,60})\\s*$', 'inline'),
    'C1-byline-alone': ("^[Bb]y\\s+[A-Z][\\w.'-]+(?:\\s+[A-Z][\\w.'-]+){0,3}$", 'anywhere'),
    'C1-byline': ("(?i)^(?:by\\s+[A-Z][\\w.'-]+(?:\\s+[A-Z][\\w.'-]+){0,3}\\b.*|from:\\s.*|posted:?\\s.*|updated\\s.*|published\\s.*|contributed by\\s.*|written by\\s.*|latest revision\\s.*|(?:(?:Monday|Tuesday|Wednesday|Thursday|Friday|Saturday|Sunday),?\\s+)?(?:(?:January|February|March|April|May|June|July|August|September|October|November|December)\\s+\\d{1,2},?\\s+\\d{4}|\\d{1,2}\\s+(?:January|February|March|April|May|June|July|August|September|October|November|December)\\s+\\d{4})\\.?)$", 'start'),
    'C1-credit': ("(?i)^\\W{0,3}(?:©|copyright\\b|\\(c\\)\\s*\\d{4}|source:|article source:|website:|courtesy:|photo:|image:|editor'?s note:|editor:|correction|published\\s+\\d{1,2}\\s)", 'anywhere'),
    'C1-navigation': ('^(?=.{0,160}$)(?:[^|:»\\n]+\\s(?:\\||:|»)\\s){3,}[^|:»\\n]+$|^.*»\\s*\\|\\|.*»\\s*\\|?$', 'anywhere'),
    'C1-promotion': ("(?i)^\\W{0,3}(?:paperback:|hardcover:|publisher:|language:\\s*\\w+$|isbn-1[03]:|product dimensions:|indian edition also available|we've sent an email with your order|help support .{0,40} and get|don't forget to check out|click on the .{0,30} to follow me|subscribe to (?:our|the) newsletter)", 'anywhere'),
}
# 'anywhere': the whole line is removed if it matches and is short;
# 'start': only in the first three non-empty lines; 'inline': only the matched end of a long line.

# Line scorer (full code: c1model.py). x = the line's features, standardised with mu and sd.
# score = 1 / (1 + exp(-(sum(w * (x - mu) / sd) + b)))
THRESHOLD = 0.97
B = -3.0301
FEATURES = {  # name: (weight, mu, sd)
    'short_60': (-0.301, 0.254, 0.435),
    'short_120': (+0.371, 0.415, 0.493),
    'log_length': (-1.363, 4.974, 1.116),
    'ends_sentence': (-0.477, 0.734, 0.442),
    'first_3': (+1.104, 0.270, 0.444),
    'last_3': (+1.791, 0.288, 0.453),
    'rel_position': (-0.342, 0.510, 0.337),
    'link_words': (+1.016, 0.099, 0.347),
    'url': (+0.098, 0.007, 0.080),
    'copyright': (+0.088, 0.013, 0.113),
    'date': (-0.251, 0.094, 0.292),
    'separators': (+0.745, 0.043, 0.430),
    'capitalised': (+1.027, 0.193, 0.220),
    'bullet': (-0.289, 0.202, 0.402),
    'byline_start': (+1.035, 0.020, 0.138),
    'repeated_docs': (+0.000, 0.000, 0.000),
    'digits': (+1.155, 0.011, 0.035),
    'few_words': (-0.384, 0.068, 0.251),
    'next_is_paragraph': (-1.193, 0.420, 0.494),
    'prev_is_paragraph': (+0.046, 0.438, 0.496),
}
```

3. **Action:** fix. The matched text is removed; the rest of the document is kept.

4. **Harm test:** proxy gold set = the 59 rows of `labels.csv` with verdict `clean` (each read in full, no disease found). The medicine changes or drops **0 of 59** of them (0.0%, each tested by a scorer that had not seen it): **pass** against the 1% bar. On the whole pack it changes 150 of 500 documents and removes 13,477 of 2,442,877 characters (0.55%). The official gold set for English (owned by Data A) is not published yet; this test will be repeated on it when it is.

5. **Side effects:** on documents the scorer had not seen, the medicine finds 53 of the 86 labelled C1 lines (62%; the phrase rules alone find 47). It misses C1 that has no fixed wording and no boilerplate signal: one-off site headings ("Programs for children", "Chapter 02 Worksheet"), reference lists and citations, catalogue records and promotion inside a long paragraph. These are left to the readers. Article titles are kept on purpose: the first version of the scorer removed titles and sentences such as "By the end of 1992, …" from clean documents, which is why the scorer may only remove signalled lines. Remaining risks: a "Source:" line under a chart, and a short line with a date or a link word that belongs to the article.

## Medicine 2: C3 truncated document

1. **Disease:** C3, in 6 of the 150 labelled documents (6 labelled cases). Three examples:
   - `2067313a5e000336` (document 5): "He writes in the Guardian on Tuesday:"
   - `8620d74ba9ae6cdc` (document 24): "For more information about the collection, visit"
   - `d9cf118c7cae38e0` (document 42): "week we celebrate the holiday of Chanuka."

2. **Medicine:** Remove the last line of a document when it is an unfinished sentence: the last non-empty line has at least six words and ends on a word no complete sentence ends on (for example "the", "of", "to", "visit", "they") or on ":" or ",", and the C1 medicine does not already remove it. The document then ends at its last complete line. Run after the C1 medicine. Two broader rules were tested and rejected (25 Oct): "the last line ends with any lower-case letter" also removed titles and author notes at the end of good documents, and "the text starts with a lower-case letter" also removed dictionary entries that start in lower case. Those cases stay with the readers.

   In code (Python):

```python
DANGLING = {'a', 'about', 'an', 'and', 'are', 'as', 'at', 'be', 'because', 'been', 'but', 'by', 'can', 'could', 'for', 'from', 'had', 'has', 'have', 'he', 'her', 'his', 'i', 'if', 'in', 'including', 'into', 'is', 'it', 'its', 'more', 'most', 'not', 'of', 'on', 'or', 'our', 'see', 'she', 'such', 'than', 'that', 'the', 'their', 'these', 'they', 'this', 'those', 'to', 'very', 'visit', 'was', 'we', 'were', 'when', 'which', 'while', 'who', 'whose', 'will', 'with', 'would', 'you', 'your'}

def c3_last_line(text, c1_removed_lines):
    lines = [l for l in text.split('\n') if l.strip()]
    if not lines or lines[-1] in c1_removed_lines:
        return None
    words = lines[-1].rstrip().split()
    if len(words) >= 6 and (words[-1][-1] in ':,' or words[-1].lower() in DANGLING):
        return lines[-1]   # remove this line
    return None
```

3. **Action:** fix. The matched text is removed; the rest of the document is kept.

4. **Harm test:** proxy gold set = the 59 rows of `labels.csv` with verdict `clean` (each read in full, no disease found). The medicine changes or drops **0 of 59** of them (0.0%): **pass** against the 1% bar. On the whole pack it changes 3 of 500 documents and removes 268 of 2,442,877 characters (0.01%). The official gold set for English (owned by Data A) is not published yet; this test will be repeated on it when it is.

5. **Side effects:** the medicine finds 1 of the 6 labelled cases. It misses cut-offs that do not end on a dangling word or a colon (for example a teaser ending in "..."), and documents that start in the middle of a sentence. Some labelled cut-offs are removed by the C1 medicine first, because their last line is also a link ("…For more information about the collection, visit"). On the whole pack it changes 3 documents; checked on 25 Oct, one of them is a title ending in "On" ("The Saga Lives On"), which is a wrong removal.
