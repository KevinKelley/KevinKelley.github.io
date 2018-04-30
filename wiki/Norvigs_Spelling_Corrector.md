---
layout: default
---

This is a side-trip in the [Monads](/wiki/Monads.html) pages, here because it illustrates List Comprehensions with an interesting bit of code.

With little discussion, this is Norvig's spelling corrector in three versions: Python, Clojure, and Fantom.

[...in Python]
==============
[source](http://norvig.com/spell-correct.html)
```
import re, collections

def words(text): return re.findall('[a-z]+', text.lower())

def train(features):
    model = collections.defaultdict(lambda: 1)
    for f in features:
        model[f] += 1
    return model

NWORDS = train(words(file('big.txt').read()))

alphabet = 'abcdefghijklmnopqrstuvwxyz'

def edits1(word):
   splits     = [(word[:i], word[i:]) for i in range(len(word) + 1)]
   deletes    = [a + b[1:] for a, b in splits if b]
   transposes = [a + b[1] + b[0] + b[2:] for a, b in splits if len(b)>1]
   replaces   = [a + c + b[1:] for a, b in splits for c in alphabet if b]
   inserts    = [a + c + b     for a, b in splits for c in alphabet]
   return set(deletes + transposes + replaces + inserts)

def known_edits2(word):
    return set(e2 for e1 in edits1(word) for e2 in edits1(e1) if e2 in NWORDS)

def known(words): return set(w for w in words if w in NWORDS)

def correct(word):
    candidates = known([word]) or known(edits1(word)) or known_edits2(word) or [word]
    return max(candidates, key=NWORDS.get)
```

...in Clojure
=============
[source](http://en.wikibooks.org/wiki/Clojure_Programming/Examples/Norvig_Spelling_Corrector)
```
(defn words [text] (re-seq #"[a-z]+" (.toLowerCase text)))

(defn train [features]
  (reduce (fn [model f] (assoc model f (inc (get model f 1)))) {} features))

(def *nwords* (train (words (slurp "big.txt"))))

(defn edits1 [word]
  (let [alphabet "abcdefghijklmnopqrstuvwxyz", n (count word)]
    (distinct (concat
      (for [i (range n)] (str (subs word 0 i) (subs word (inc i))))
      (for [i (range (dec n))]
        (str (subs word 0 i) (nth word (inc i)) (nth word i) (subs word (+ 2 i))))
      (for [i (range n) c alphabet] (str (subs word 0 i) c (subs word (inc i))))
      (for [i (range (inc n)) c alphabet] (str (subs word 0 i) c (subs word i)))))))

(defn known [words nwords] (seq (for [w words :when (nwords w)]  w)))

(defn known-edits2 [word nwords] (seq (for [e1 (edits1 word) e2 (edits1 e1) :when (nwords e2)]  e2)))

(defn correct [word nwords]
  (let [candidates (or (known [word] nwords) (known (edits1 word) nwords)
                       (known-edits2 word nwords) [word])]
    (apply max-key #(get nwords % 1) candidates)))
```

Norvig's Spelling Corrector in Fantom
===========================
```
class Corrector
{
  Str:Int dict := train(words(File(`big.txt`).readAllStr))

  Str alphabet := ('a'..'z').toList.map(|Int ch->Str|{ch.toChar}).join  // :-)

  static Str[] words(Str text) { Regex.fromStr("[^a-z]+").split(text.lower) }

  static Str:Int train(Str[] features, Str:Int model := Str:Int[:])
  {
    features.each |word| { model[word] = model[word]?.plus(1) ?: 1 }
    return model
  }

  Str[] edits1(Str word)
  {
    n := word.size; if (n<2) return [,]
    set := Str[,]
    (0..n-1).each|i| { set.add(word.slice(0..&lt;i) + word.slice(i+1..-1)) }
    (0..n-2).each|i| { set.add(word.slice(0..&lt;i) + word[i+1].toChar + word[i].toChar + word.slice(i+2..-1)) }
    (0..n-1).each|i| { alphabet.each|c| {set.add(word.slice(0..&lt;i) + c.toChar + word.slice(i+1..-1))} }
    (0..n  ).each|i| { alphabet.each|c| {set.add(word.slice(0..&lt;i) + c.toChar + word.slice(i..-1))} }
    return set
  }

  Str[] known_edits2(Str word)
  {
    edits := Str[,]
    edits1(word).each |e1|
    { edits1(e1).each |e2| { if (dict.containsKey(e2)) edits.add(e2) } }
    return edits
  }

  Str[] known(Str[] words) { words.findAll |wd| { dict.containsKey(wd) } }

  Str[] candidates(Str word) {
    known([word])
    .union(known(edits1(word)))
    .union(known_edits2(word))
    .add(word)
  }

  Str correct(Str word) { candidates(word).max|a,b| { dict[a] &lt;=> dict[b] } }

  Void main()
  {
    sentence := "wnen n tha curse uv wijet evenz it bekoms necesary"
    echo(sentence)
    echo( words(sentence) .map(|Str wd->Str|{correct(wd)}) .join(" "))
  }
}
```
