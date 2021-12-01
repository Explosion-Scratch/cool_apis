# Cool apis
A collection of implementations to scrape APIs of popular websites.

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

### Get parts of speech of text
```js
/**
* Gets parts of speech for a sentence
* @param {String} text The text to get parts of speech for.
* @returns {Promise.<Object>} Resolves into a list of parts of speech. (Or rejects with an error)
* @example 
* var parts_of_speech  = await parts_of_speech("Sometimes I just want to code in JavaScript all day.");
* // ⇒
* // {entities: Array(1), sentiments: Array(1), documentInfo: {…}, wordFreq: Array(4), taggedText: '<span class="tag ADV">Sometimes</span> <span class…>all</span> <span class="tag DURATION">day</span>'}
function parts_of_speech(text) {
*/
function parts_of_speech(text) {
  return new Promise(async (resolve, reject) => {
    fetch("https://showcase-serverless.herokuapp.com/pos-tagger", {
      headers: {
        accept: "application/json",
        "content-type": "application/json",
      },
      body: JSON.stringify({ sentence: text }),
      method: "POST",
    })
      .then((res) => res.json())
      .then(resolve)
      .catch(reject);
  });
}
```

### Get quizlet flashcards
```js
/**
* Gets a list of terms from a quizlet set
* @param {String} id The id of the quizlet set.
* @returns {Promise.<String[]>} Resolves into a list of terms and definitions. (Or rejects with an error)
* @example 
* var terms  = await quizlet("213648175");
*/
async function quizlet(id){
    let res = await fetch(`https://quizlet.com/webapi/3.4/studiable-item-documents?filters%5BstudiableContainerId%5D=${id}&filters%5BstudiableContainerType%5D=1&perPage=5&page=1`).then(res => res.json())
    let currentLength = 5;
    let token = res.responses[0].paging.token
    let terms = res.responses[0].models.studiableItem;
    let page = 2;
    console.log({token, terms})
    while (currentLength >= 5){
        let res = await fetch(`https://quizlet.com/webapi/3.4/studiable-item-documents?filters%5BstudiableContainerId%5D=${id}&filters%5BstudiableContainerType%5D=1&perPage=5&page=${page++}&pagingToken=${token}`).then(res => res.json());
        terms.push(...res.responses[0].models.studiableItem);
        currentLength = res.responses[0].models.studiableItem.length;
        token = res.responses[0].paging.token;
    }
    return terms;
}
```

### Grammar check API
```js
async function grammar(text){
	let res = await fetch("https://services.gingersoftware.com/Ginger/correct/jsonSecured/GingerTheTextFull?callback=$&text=They+dont+knowledge+where+tom+waent&apiKey=GingerWebsite&clientVersion=2.0&lang=US").then(res => res.text())
	return JSON.parse(res.replace(/^$\(/, "").replace(/);?$/, ""));//Returns a 'callback'
}
```

### Google autocomplete API
```js
async function autocomplete(text){
	return JSON.parse(await fetch("https://www.google.com/complete/search?q=testing&client=Firefox").then(res => res.text()))[1];
}
```

### Search CDNJS
```js
async function cdnjs(query){
	return await fetch("https://2qwlvlxzb6-2.algolianet.com/1/indexes/*/queries", {
		headers: {
		  accept: "*/*",
		  "content-type": "application/x-www-form-urlencoded",
		  "x-algolia-api-key": `2663c73014d2e4d6d1778cc8ad9fd010`,
		  "x-algolia-application-id": `2QWLVLXZB6`,
		},
		body: JSON.stringify({
		  requests: [{ indexName: "libraries", params: e`query=${q}` }],
		}),
		method: "POST",
	      })
		.then((res) => res.json())
		.then((res) => {
		  let results = res.results[0].hits;
		  return results.map((item) => ({
		    name: item.name,
		    keywords: item.keywords,
		    snippet: item.description,
		    author: item.author,
		  }));
		});
}
