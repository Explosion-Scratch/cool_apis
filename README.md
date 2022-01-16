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
async function quizlet(id, cors = 'https://cors.explosionscratc.repl.co/'){
    let res = await fetch(`${cors}quizlet.com/webapi/3.4/studiable-item-documents?filters%5BstudiableContainerId%5D=${id}&filters%5BstudiableContainerType%5D=1&perPage=5&page=1`).then(res => res.json())
    let currentLength = 5;
    let token = res.responses[0].paging.token
    let terms = res.responses[0].models.studiableItem;
    let page = 2;
    console.log({token, terms})
    while (currentLength >= 5){
        let res = await fetch(`${cors}quizlet.com/webapi/3.4/studiable-item-documents?filters%5BstudiableContainerId%5D=${id}&filters%5BstudiableContainerType%5D=1&perPage=5&page=${page++}&pagingToken=${token}`).then(res => res.json());
        terms.push(...res.responses[0].models.studiableItem);
        currentLength = res.responses[0].models.studiableItem.length;
        token = res.responses[0].paging.token;
    }
    return terms;
}
```

### Grammar check API
```js
/**
 * Checks the grammar of text using the Ginger grammar checker API
 * @param {String} text The text to check
 * @returns {Promise.<Object>}
 * @example
 *  // ⇒ {
 *  //   "Corrections": [...],
 *  //   "Sentences": [...]
 *  //}
 */
async function grammar(text){
	let res = await fetch(`https://services.gingersoftware.com/Ginger/correct/jsonSecured/GingerTheTextFull?callback=$&text=${encodeURIComponent(text)}&apiKey=GingerWebsite&clientVersion=2.0&lang=US`).then(res => res.text())
	return JSON.parse(res.replace(/^\$\(/, "").replace(/\);?$/, ""));//Returns a 'callback'
}
```

### Another grammar checker API (cram.com)
```js
/**
* Checks the grammar of some text using the cram.com API. 
* NOTE: There is a minimum word limit of 10 words.
* @param {String} text The text to grammar check
* @example
* await checkGrammar("five words are not enough");
* // ⇒ {
* //  "meta": {
* //      "statusCode": 400
* //  },
* //  "errors": [
* //      {
* //          "status": 400,
* //          "detail": "Minimum word count criteria did not match!"
* //      }
* //  ]
* //}
* 
* await checkGrammar("Thhis word is mispelled and I dont care. The sentence also has other, grammar, errorz")
* // ⇒ [Response too long, see https://gist.github.com/Explosion-Scratch/7cae63ec6c9315c4eabcb8e27818a1e6]
*/
function checkGrammar(text){
    const opts = {
        headers: {
            "accept": "application/vnd.splat+json",
        }
    }
    return new Promise(async (resolve, reject) => {
        let res = await fetch("https://api.cram.com/article-evaluations", {
          "body": JSON.stringify({
              "text": text,
              "evaluate":["grammar","plagiarism"]
          }),
          "method": "POST",
          ...opts,
        });
        if (res.status !== 200){
          return reject(await res.json())
        } else {
          res = await res.json();
        }
        let int = setInterval(async () => {
            let res = await fetch(`https://api.cram.com/article-evaluations/${job.data.id}`, {
              "headers": {
                "accept": "application/vnd.splat+json",
              },
              "method": "GET",
            }).then(res => res.json())
            if (res.data.is_success === 1){
                clearInterval(int);
                return resolve(res.data.result);
            }
        }, 500)
    });
}
```

### Google autocomplete API
```js
/**
* Autocompletes text using Google's autocomplete engine
* @param {String} text The text to autocomplete
*/
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
```

### Search a notion space
```js
/**
* Searches a notion space for a query
* @param {String} thing The search query
* @param {String} space The notion space ID to search (localStorage.ajs_group_id on notion)
* @returns {Promise.<Object[]>} Returns a promise that resolves into an array of results
*/
async function search(thing, space, cors = `https://cors.explosionscratc.repl.co/`){
    console.log(id(space));
    let body = {
        type: "BlocksInSpace",
        query: thing,
        spaceId: id(space),
        limit: 5,
        filters: {
            isDeletedOnly: false,
            excludeTemplates: false,
            isNavigableOnly: false,
            requireEditPermissions: false,
            ancestors: [],
            createdBy: [],
            editedBy: [],
            lastEditedTime: {},
            createdTime: {},
        },
        sort: "Relevance",
        source: "quick_find",
    };
    let {results} = await fetch(`${cors}www.notion.so/api/v3/search`, {
        headers: {"Content-Type": "application/json; charset=UTF-8"},
        body: JSON.stringify(body),  
        method: "POST",
    }).then(res => res.json());
    return results;

    function id(notactualuuid) {
        let lengths = [8, 4, 4, 4, 12];
        let re = new RegExp(lengths.map(i => `([a-zA-F0-9]{${i}})`).join(""));
        return notactualuuid.match(re).slice(1).join("-")
    }
}
```

### Convertio API
```js
/* Usage */
/**
* Converts a file to a given format
* @param {File} file A JS file object to convert
* @param {Object} opts The options object
* @param {String} [opts.content=base64Encode(file)] The base64 encoded file content to convert. Inferred from the file object passed
* @param {String} opts.output The output format to convert to
* @param {Array.<String>} opts.apiKeys The apiKeys to convert with (selected randomly from the array)
* @param {String} [opts.filename=file.name] The name of the file. Inferred from the name of the file passed.
*/
function convert(file, opts){
  return new Promise((resolve, reject) => {
    let reader = new FileReader();
    reader.readAsDataURL(file);
    await new Promise(res => (reader.onload = res));
    let content = reader.result.split(";base64,")[1];
    let conversion = new Conversion({filename: file.name, content, output: type, ...opts});
    conversion.run().then(resolve).catch(reject);
  });
}

class Conversion {
    constructor({content, filename, output, apiKeys}) {
        Object.assign(this, {
            content,
            filename,
            output,
            apiKeys,
        });
    }
    async run() {
        await this.start();
        return new Promise(res=>{
            let int = setInterval(async()=>{
                try {
                await this.getStatus();
                } catch(e){throw e};
                if (this.url) {
                    window.clearInterval(int);
                    res(this.url);
                }
            }
            , 1000);
        }
        )
    }
    async start() {
        let keys = this.apiKeys;
        if (!this.apiKeys?.length){
          throw new Error("No convertio api keys (use when initiating, get for free at https://developers.convertio.co/)");
        }
        let result = await fetch("https://api.convertio.co/convert", {
            body: JSON.stringify({
                "apikey": keys[Math.floor(Math.random() * keys.length)],
                "input": "base64",
                "file": this.content,
                "filename": this.filename,
                "outputformat": this.output,
            }),
            headers: {
                "Content-Type": "application/json"
            },
            method: "POST"
        }).then(res=>res.json());
        if (result.error){
            throw new Error(result.error);
        }
        this.id = result.data.id;
        return result;
    }
    async getStatus() {
        let result = await fetch(`https://api.convertio.co/convert/${this.id}/status`).then(res=>res.json());
        this.status = result;
        if (result.status === "ok" && result.data.step === "finish") {
            this.url = result.data.output.url;
            this.output = result.data.output;
        }
        if (result.error){
            throw new Error(result.error);
        }
        return result;
    }
}
```

### Sci-hub scraping

```js
/**
* Gets the PDF URL from a scientific article's DOI and a sci-hub domain
* @param {String} doi A scientific article's DOI (Digital Object Identifier System)
* @param {String} domain The hostname of a sci-hub instance.
* @param {String} cors The cors proxy to use
* @returns {Promise.<string>} Returns a promise which resolves into a URL of the PDF
* @example
*/
async function getURL(doi, domain = 'sci-hub.st', cors = 'https://cors.explosionscratc.repl.co/') {
  let text = await fetch(`${cors}${domain}/${doi}`).then(res => res.text())
  return new DOMParser().parseFromString(text, "text/html").querySelector("iframe, embed").src
}
```