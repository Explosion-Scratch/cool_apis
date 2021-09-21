# Cool apis
A collection of implementations to scrape APIs of popular websites.

# Text processing
Translation, rewriting and (soon) more!

### Translate text (translate.google.com)
```js
/**
 * Translates text to a certain language.
 * @param {String} text The text to translate (or an options object)
 * @param {String} target The target language to translate to.
 * @param {String} source The source language.
 * @returns {Promise.<object>} Returns a promise resolving into an object with the translated text, raw response JSON, the original text, and the target and source languages.
 * @example
 * var translated = await translate("Hello world", "fr");
 * // ⇒ 
 * // {
 * //   source: "en", 
 * //   original: "Hello world",
 * //   translated: "Bonjour le monde",
 * //   result: "weird google stuff here"
 * // }
 *
 */
async function translate(text, target, source, proxy) {
  if (typeof text == "object") {
    target = text.target;
    source = text.source;
    proxy = text.proxy;
    text = text.text;
  }
  var opts = {
    text: text || "",
    source: source || 'auto',
    target: target || "en",
    proxy: proxy || "",
	}
  var result = await fetch(
    `https://${opts.proxy}translate.googleapis.com/translate_a/single?client=gtx&sl=${opts.source}&tl=${opts.target}&dt=t&q=${encodeURI(opts.text)}`
  ).then(res => res.json());
  return {
    source: opts.source,
    target: opts.target,
    original: text,
    translated: result[0]?.[0]?.[0],
    result,
  };
}
```

### Rewrite text
```js
/**
* Rewrites text
* @param {String} text The text to rewrite.
* @returns {Promise.<String[]>} Resolves into a list of about 10 rewritten versions. (Or rejects with an error)
* @example 
* var rewritten  = await rewrite("Sometimes I just want to code in JavaScript all day.");
* // ⇒ [
* //    "I sometimes just want to code JavaScript all day.",
* //    "JavaScript is my favorite programming language sometimes.",
* //    "I sometimes wish I could code in JavaScript all day.",
* //    "JavaScript is sometimes all I want to do all day.",
* //    "I like JavaScript sometimes and want to code all day long.",
* //    "Some days I just want to work all day in JavaScript.",
* //    "It's not uncommon for me to just want to code in JavaScript all day.",
* //    "My favorite thing to do sometimes is just code JavaScript all day.",
* //    "My favourite coding language is JavaScript, which I can code all day.",
*//     "JavaScript is my preferred language sometimes, since it lets me code all day.",
*// ];
*/
function rewrite(text) {
  return new Promise(async (resolve, reject) => {
    var { suggestions, error_code, error_msg, error_msg_extra } = await fetch(
      "https://api.wordtune.com/rewrite-limited",
      {
        headers: {
          accept: "*/*",
          "accept-language": "en-US,en;q=0.9",
          "content-type": "application/json",
          "x-wordtune-origin": "https://www.wordtune.com",
        },
        referrer: "https://www.wordtune.com/",
        body: JSON.stringify({
          action: "REWRITE",
          text: text,
          start: 0,
          end: text.length,
          selection: {
            wholeText: text,
            start: 0,
            end: text.length,
          },
        }),
        method: "POST",
      }
    ).then((res) => res.json());
    if (error_code || error_msg || error_msg_extra) {
      reject({
        code: error_code,
        message: error_msg,
        message_extra: error_msg_extra,
      });
    } else {
      resolve(suggestions);
    }
  });
}
```

### Get parts of speech of some text
```js
function parts_of_speech(text) {
  return new Promise(async (resolve, reject) => {
    fetch("https://showcase-serverless.herokuapp.com/pos-tagger", {
      headers: {
        accept: "application/json",
        "content-type": "application/json",
      },
      body: JSON.stringify({ sentence: "Hello world" }),
      method: "POST",
    })
      .then((res) => res.json())
      .then(resolve)
      .catch(reject);
  });
}
```
