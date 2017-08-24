OpenNMT provides generic tokenization utilities to quickly process new training data.

!!! note "Note"
    For LuaJIT users, tokenization tools require the `bit32` package.

## Tokenization

To tokenize a corpus:

```bash
th tools/tokenize.lua OPTIONS < file > file.tok
```

## Detokenization

If you activate `-joiner_annotate` marker, the tokenization is reversible. Just use:

```bash
th tools/detokenize.lua OPTIONS < file.tok > file.detok
```

## Special characters

* `￨` is the feature separator symbol. If such character is used in source text, it is replaced by its non presentation form `│`.
* `￭` is the default joiner marker (generated in `-joiner_annotate marker` mode). If such character is used in source text, it is replaced by its non presentation form `■`

## Mixed casing words
`-segment_case` feature enables tokenizer to segment words into subwords with one of 3 casing types (truecase ('House'), uppercase ('HOUSE') or lowercase ('house')), which helps  restore right casing during  detokenization. This feature is especially useful for texts with a signficant number of words with mixed casing ('WiFi' -> 'Wi' and 'Fi').
```text
WiFi --> wi￨C fi￨C
TVs --> tv￨U s￨L
```

## Alphabet Segmentation
Two options provide specific tokenization depending on alphabet:

* `-segment_alphabet_change`: tokenize a sequence between two letters when their alphabets differ - for instance between a Latin alphabet character and a Han character.
* `-segment_alphabet Alphabet`: tokenize all words of the indicated alphabet into characters - for instance to split a chinese sentence into characters, use `-segment_alphabet Han`:

```text
君子之心不胜其小，而气量涵盖一世。 --> 君 子 之 心 不 胜 其 小 ， 而 气 量 涵 盖 一 世 。
```

## BPE

OpenNMT's BPE module fully supports the [original BPE](https://github.com/rsennrich/subword-nmt) as default mode:

```bash
tools/tokenize.lua TOK_OPTIONS < input > input_tokenized
tools/learn_bpe.lua -size 30000 -save_bpe codes < input_tokenized

# The same TOK_OPTIONS should be applied as the pre-tokenization for BPE model training
# Additional JOINNER_OPTIONS can be used ONLY at this stage, enabling detokenization
tools/tokenize.lua TOK_OPTIONS JOINER_OPTIONS -bpe_model codes < input
```

with two additional features:

**1\. Add support for different modes of handling prefixes and/or suffixes: `-bpe_mode`**

* `suffix`: BPE merge operations are learnt to distinguish sub-tokens like "ent" in the middle of a word and "ent<\w>" at the end of a word. "<\w>" is an artificial marker appended to the end of each token input and treated as a single unit before doing statistics on bigrams. This is the default mode which is useful for most of the languages.
* `prefix`: BPE merge operations are learnt to distinguish sub-tokens like "ent" in the middle of a word and "<w\>ent" at the beginning of a word. "<w\>" is an artificial marker appended to the beginning of each token input and treated as a single unit before doing statistics on bigrams.
* `both`: `suffix` + `prefix`
* `none`: No artificial marker is appended to input tokens, a sub-token is treated equally whether it is in the middle or at the beginning or at the end of a token.

**2\. Add support for BPE in addition to the case feature: `-bpe_case_insensitive`**

OpenNMT's tokenization flow first applies BPE then add the case feature for each input token. With the standard BPE, "Constitution" and "constitution" may result in the different sequences of sub-tokens:

```text
Constitution --> con￨C sti￨l tu￨l tion￨l
constitution --> consti￨l tu￨l tion￨l
```

If you want a *caseless* split so that you can take the best from using case feature, and you can achieve that with the following command lines:

```bash
tools/tokenize.lua TOK_OPTIONS < input > input_tokenized

# We don't need BPE to care about case
tools/learn_bpe.lua -size 30000 -lc -save_bpe codes_lc < input_tokenized

# The case information is preserved in the true case input
tools/tokenize.lua TOK_OPTIONS JOINER_OPTIONS -bpe_model codes_lc < input
```
!!! note "Note"
    If the BPE model is learnt by the original python learn_bpe.py, use `-bpe_case_sensitive` option to the above command line and make sure that this model are learnt over lowercased train data. This option is automatically overridden, thus has no effect, if the BPE model is learnt by OpenNMT style learn_bpe.lua.

The output of the previous example would be:

```text
Constitution --> con￨C sti￨l tu￨l tion￨l
constitution --> con￨l sti￨l tu￨l tion￨l
```

!!! note "Note"
    Use Lua 5.2 if you encounter any memory issue while using `learn_bpe.lua` (e.g. `-size` is too big). Otherwise, stay with LuaJIT for better efficiency.
