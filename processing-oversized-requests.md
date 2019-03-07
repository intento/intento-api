# Processing oversized requests

Each API provider has its own limits on the size of the request. These limits vary a lot across different providers, but most of the cases refer to one of the following:

- **max-item-length** is a maximum size of text segment in a request.
- **max-array-length** is a maximum number of segments in array in a request.
- **max-total-length** is a maximum total length of all segments in a request.
- **max-request-length** is the maximum total size of a request in bytes.

To process data which hits any of those limits, you may either split it into a number of requests by yourself or enable the [Intento async mode](https://github.com/intento/intento-api#async-mode), sit back and relax.

In the async mode, if the request fits in all the limits of the selected API provider, it is sent to the provider in a single request, much like it happens in the synchronous mode.

If the request contains an array which violates any of these three limits: max-array-length, max-request-length or max-total-length, we split it into multiple small sub-arrays. Those sub-arrays are sent to the provider concurrently and merged when the async job is completed.

Another situation is when the original request exceeds max-item-length. It may be a single large text segment (e.g. html or xml) that breaks limits or any number of such large segments in the array. We use a specific text segmentation tool to split those large segments into smaller ones, concurrently send them for the processing and then compile results in the original order.

Depending on the input format we choose a suitable tool for cutting large segments. We support valid splitting for such formats: TEXT, HTML/XML. Let's take a closer look at each one.

## SPLIT

### TEXT split

The TEXT format splitting is always applied, except when we have a particular format splitting.

#### TEXT overview

For the text format, we use [nltk.sent_tokenize](https://www.nltk.org/api/nltk.tokenize.html) which splits chunks of text into sentences. If a sentence still exceeds the provider's limit, then we split it by a space, a line break or a tabulation: `\s\n\t`.

### HTML/XML split

If the provider supports the HTML/XML format, then we use this type of splitting.

#### HTML/XML overview

For the HTML/XML format, we transform an HTML document into a tree, where each tag is a node. Then we traverse the tree in depth until there is a node which doesn't exceed the provider's limits. We send this tag for translation as a separate item or as an element in an array of other items (depends on the provider's support of [bulk requests](https://github.com/intento/intento-api/blob/master/ai.text.translate.md#bulk-mode)). And finally, we insert the translated element back into the initial HTML document.

#### Important notes

- The HTML/XML split fixes invalid HTML with missing opening or closing tags by adding missing one.
- We don't send for translation content of these tags:
    - `area`
    - `basefont`
    - `datalist`
    - `link`
    - `meta`
    - `noembed`
    - `noframes`
    - `param`
    - `rp`
    - `script`
    - `source`
    - `style`
    - `template`
    - `track`
    - `noscript`

Let's have a look how it works. We have an HTML input:

```html
<html>
<head>
<title>This is title</title>
<meta content="text/html; charset=utf-8" http-equiv="Content-Type"/>
</head>
<body>
<p>The young Princess Bolkónskaya had brought some work in a gold-embroidered velvet bag.<br> Her pretty little upper lip, on which a delicate dark down was just perceptible, was too short for her teeth, but it lifted all the more sweetly, and was especially charming when she occasionally drew it down to meet the lower lip. <br>As is always the case with a thoroughly attractive woman, her defect&mdash;the shortness of her upper lip and her half-open mouth&mdash;seemed to be her own special and peculiar form of beauty. Everyone brightened at the sight of this pretty young woman, so soon to become a mother, so full of life and health, and carrying her burden so lightly. <br>Old men and dull dispirited young ones who looked at her, after being in her company and talking to her a little while, felt as if they too were becoming, like her, full of life and health. All who talked to her, and at each word saw her bright smile and the constant gleam of her white teeth, thought that they were in a specially amiable mood that day.</p>
<p>The little princess went round the table with quick, short, <span>swaying steps, her workbag on her arm, and gaily spreading out her dress sat</span> down on a sofa near the silver samovar, as if all she was doing was a pleasure to herself and to all around her. &ldquo;I have brought my work,&rdquo; said she in <i>French</i>, displaying her bag and addressing all present. &ldquo;Mind, Annette, I hope you have not played a wicked trick on me,&rdquo; she added, turning to her hostess. &ldquo;You wrote that it was to be quite a small reception, and just see how badly I am dressed.&rdquo; And she spread out her arms to show her short-waisted, lace-trimmed, dainty gray dress, girdled with a broad ribbon just below the breast.</p>
</body>
</html>
</body>
</html>
```

Splitting turns HTML/XML input into an array. How we break strings for translations depends on the limits of the provider. One of the possible cases for our example is when the input doesn't exceed limits, and we send it without splitting. Another split case is when the entire input does not pass along the limits, but the parts do. Then the request for translation will be:

```python
[
'<title>This is title</title>',
'<body>\n<p>The young Princess Bolkónskaya had brought some work in a gold-embroidered velvet bag.<br> Her pretty little upper lip, on which a delicate dark down was just perceptible, was too short for her teeth, but it lifted all the more sweetly, and was especially charming when she occasionally drew it down to meet the lower lip. <br>As is always the case with a thoroughly attractive woman, her defect—the shortness of her upper lip and her half-open mouth—seemed to be her own special and peculiar form of beauty. Everyone brightened at the sight of this pretty young woman, so soon to become a mother, so full of life and health, and carrying her burden so lightly. <br>Old men and dull dispirited young ones who looked at her, after being in her company and talking to her a little while, felt as if they too were becoming, like her, full of life and health. All who talked to her, and at each word saw her bright smile and the constant gleam of her white teeth, thought that they were in a specially amiable mood that day.</p>\n<p>The little princess went round the table with quick, short, <span>swaying steps, her workbag on her arm, and gaily spreading out her dress sat</span> down on a sofa near the silver samovar, as if all she was doing was a pleasure to herself and to all around her. “I have brought my work,” said she in <i>French</i>, displaying her bag and addressing all present. “Mind, Annette, I hope you have not played a wicked trick on me,” she added, turning to her hostess. “You wrote that it was to be quite a small reception, and just see how badly I am dressed.” And she spread out her arms to show her short-waisted, lace-trimmed, dainty gray dress, girdled with a broad ribbon just below the breast.</p>\n</body>'
]
```

Other case is when the content of the body tag does not pass the limits. Then we traverse tag structure in depth until we have an element which doesn't exceed the provider's limits. Next possible splitting is when the content of p tags passes limits:

```python
[
'<title>This is title</title>',
'<p>The young Princess Bolkónskaya had brought some work in a gold-embroidered velvet bag.<br> Her pretty little upper lip, on which a delicate dark down was just perceptible, was too short for her teeth, but it lifted all the more sweetly, and was especially charming when she occasionally drew it down to meet the lower lip. <br>As is always the case with a thoroughly attractive woman, her defect—the shortness of her upper lip and her half-open mouth—seemed to be her own special and peculiar form of beauty. Everyone brightened at the sight of this pretty young woman, so soon to become a mother, so full of life and health, and carrying her burden so lightly. <br>Old men and dull dispirited young ones who looked at her, after being in her company and talking to her a little while, felt as if they too were becoming, like her, full of life and health. All who talked to her, and at each word saw her bright smile and the constant gleam of her white teeth, thought that they were in a specially amiable mood that day.</p>',
'<p>The little princess went round the table with quick, short, <span>swaying steps, her workbag on her arm, and gaily spreading out her dress sat</span> down on a sofa near the silver samovar, as if all she was doing was a pleasure to herself and to all around her. “I have brought my work,” said she in <i>French</i>, displaying her bag and addressing all present. “Mind, Annette, I hope you have not played a wicked trick on me,” she added, turning to her hostess. “You wrote that it was to be quite a small reception, and just see how badly I am dressed.” And she spread out her arms to show her short-waisted, lace-trimmed, dainty gray dress, girdled with a broad ribbon just below the breast.</p>\n</body>'
]
```

Let's assume that we have a provider who cannot accept such segments as input because of the low limits. So we have to traverse content of p segments. This content is a text with formatting tags. For this case we use previously mentioned TEXT split. Possible array of segments:

```python
[
'<title>This is title</title>',


'The young Princess Bolkónskaya had brought some work in a gold-embroidered velvet bag.<br> Her pretty little upper lip, on which a delicate dark down was just perceptible, was too short for her teeth, but it lifted all the more sweetly, and was especially charming when she occasionally drew it down to meet the lower lip.<br>',


'As is always the case with a thoroughly attractive woman, her defect&mdash;the shortness of her upper lip and her half-open mouth&mdash;seemed to be her own special and peculiar form of beauty. Everyone brightened at the sight of this pretty young woman, so soon to become a mother, so full of life and health, and carrying her burden so lightly.<br>',


'Old men and dull dispirited young ones who looked at her, after being in her company and talking to her a little while, felt as if they too were becoming, like her, full of life and health. All who talked to her, and at each word saw her bright smile and the constant gleam of her white teeth, thought that they were in a specially amiable mood that day.',


'The little princess went round the table with quick, short, <span>swaying steps, her workbag on her arm, and gaily spreading out her dress sat</span> down on a sofa near the silver samovar, as if all she was doing was a pleasure to herself and to all around her.',
'&ldquo;I have brought my work,&rdquo; said she in <i>French</i>, displaying her bag and addressing all present. &ldquo;Mind, Annette, I hope you have not played a wicked trick on me,&rdquo; she added, turning to her hostess.',


'&ldquo;You wrote that it was to be quite a small reception, and just see how badly I am dressed.&rdquo; And she spread out her arms to show her short-waisted, lace-trimmed, dainty gray dress, girdled with a broad ribbon just below the breast.'
]
```

Please note that HTML/XML input is correctly processed in async mode by all providers. This is true even for the providers who don't support HTML/XML in sync mode. More on the support of formatted inputs you can read in the [Intento API docs](https://github.com/intento/intento-api/blob/master/ai.text.translate.md#supported-formats-in-async-mode).

## Fulfilling requests

After splitting the input, the next stage is fulfilling requests. But occasionally during fulfilling requests, one of the subrequests may fail due to sporadic or persistent errors on the provider API. In such a case, we make three retries for each such subrequest. If it still failed, the whole large request is returned with the “failed” status.
