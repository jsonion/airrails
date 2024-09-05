## LeapGest whitepaper appendix: JSObjects vs inArrayType sorted schema
... inArrayType was developed for purposes of in-house research (what we need to make an informed choice)

_Hereby I am working on my `jsonion` framework, applying it on my side-project as seen in code blocks. I try to resolve difficulties with unauthored source or even peer-to-peer data nodes, and so I need to use proper approaches to building codebase. I am basing my claims in text in real-life examples as I develop my codebase. The published code in this document is oftentimes `just` pseudocode and sometimes very polished `JS` and `JSON`._

Lets start by making a simple claim, that JS object notation is trivial to use when naming collections of paths and values as part of the coding experience. But! These objects aren't immutable in their structure and will encounter fluctuations in key order early on, ie. printed on next tick via console.log.

Well so what, right? Were there static schema objects to allow for public checksum validation of document nodes, as received from one remote DB[collection], or found in local persistent storage, and at that with simplistic syntax that JS is known for? Well aren't web developers the chattery types!

```javascript trickaweek.com
/*/////
   lil front-end database caching logic starter
                                          */////
(bfr=window.localStorage
 &&  window.localStorage.getItem("streaks"));

 DB.Streaks = (bfr)                   &&
              (bfr = JSON.parse(bfr)) && 
                      inArrType(bfr);
   /////     becomes executable array
 
 switch (!!DB.Streaks) {
  case  true:
   if (DB.Streaks.invalidateCache < Date.now()
                                 .getSeconds())
      { localStorage.removeItem("streaks"); 
        break }
/*/////
   in other case a collection need be fetched
                                          */////
  case  false:
   if   (bfr = await getStreaks())
        {
         let invalidateCache 
          =  Date.now().getSeconds();

         DB.Streaks
          = bfr
          = inArrType(bfr, {invalidateCache});

         localStorage.setItem("streaks", 
                 JSON.stringify( bfr ));
        }
  }

    //
   //  notice JSON methods are the 1st and only
  // … to touch JSObjects (key order isn't in) 

  async function getStreaks () {
    let res = await fetchAPI("streaks", "/now");
    return res;
  }

  function fetchAPI (route, urlMix,
                           postData=null) {
   const apiRoot
     =   process && process.ENV.apiRoot
                 ||      window.apiRoot;
                /// ENV variable
   const routes
     = {
       auth: "/auth",
      users: "/users",
    streaks: "/streaks",
    };    ///  service-specific API endpoints

   const CFG = () => new Object({
      headers: {
        "Content-Type": "application/json"
      },
      mode: "cors",
      cache: "no-cache",   ////   thinking it
      redirect: "follow",
      referrerPolicy: "no-referrer",
   }),
   GET=Object.assign(CFG(), {method:"GET"}),
   POST=Object.assign(CFG(), {method:"POST"}),
   PATCH=Object.assign(CFG(), {method:"PATCH"}),
   DELETE=Object.assign(CFG(), {method:"DELETE"});

   try {
   var res = await fetch(
             apiRoot + routes[route] + urlMix, 
            (postData === null) 
             ?  GET
             :  Object.assign(POST, {
                  body: JSON.stringify(postData)
           }));
  } catch (ee) {
    throw (ee);
  }
                          ////
  if (res) return res.body &&
                  res.json();
      else return undefined;
}

/*/////
   demonstration of problem at parsing point
 … there's this lil way to tap inbound stream:

   https://developer.mozilla.org/en-US/docs/Web/API/Response/body
                                          */////

const curUser = JSON.parse(`[{
"userId": "1234", "displayName": "blaz_skate", 
                   "streakData": [
                                 {"weekIdx":36},
                                 {"weekIdx":35},
                                 {"weekIdx":34} 
                                 }]
}]`); /////  …  streaks aren't limited to sk8!

JSON.stringify(curUser);

[{"displayName":"blaz_skate","streakData":["{...}"],"userId":"1234"}]

// see key order changed for no obvious reason
// … and we couldn't even get a key-order array
```

With this example the problem does become apparent, but only for as long as there is actual need for checksum validation and schema-ordered output. And with current `API` services found at most `HTTP/S` website domains this doesn't even come close to an issue. 

Perhaps the mix on `trickaweek.com`, used as example here in the context of `jsonion`, might give us some more insight -- it's about a set of coding stages, coding problems placed with neatly packed context; there thinking `Object` and `Array` schemas is good practice anyway.

It so occurs that document nodes on inbound web traffic in browser are usually parsed using `JSON.parse()` and `response.json()`, and are thus immediately invoked stored in `JSObject`; the `Object` is soon to apply its indexing on named collections and offset the intended rigid schema before any code could intercept it to parse out its schema in array. Oh well, atleast for the limited reason of checksum validation per document node, it then really makes sense to use a schema, defined in-array (or otherwise one given `keys` order won't keep its shape at checksum). Just think about it!

The truest of reasons for this strange key disorder pattern is, under the hood, that this was so built for in-browser memory access optimization, due to transform static string paths, the hard-coded `object.key` references. No matter how big the index its access times are equal for either path in one `Object`.

Hash table indexing is employed for the optimization, memoizing shorter, static pointers to stored properties (browser links in memory addresses in applying data state to code blocks, at consumers).

There's one thing about the `Object` that wants to get discovered and I haven't yet made the realization; whether dynamic string keys receive the same treatment when a variable in bracket notation (… at any code repetition, optimized or first-time). 

So here's example of a similar object built for both front-end and backend (it can facilitate traversing paths). It creates deep object paths based on characters in words, object paths or even DOM nodes (tagged with attribute). This class can be created both with `Object` or with `Array` (like `Object.entries()`). When order of elements is crucial, for example when placing `reactive-custom-tags` in areas of template and attribute assignment, for efficient DOM path access. What is equal among both structs, `Array` or `Object`, is the tree structure. Both are versatile in their use, though `Array` is a little more difficult to implement.

```javascript trickaweek/index.html
const TRICKS = {
      stance: ["Nollie", "Switch", "Fakie", "To Fakie", "To Normal"],
      slides: ["Boardslide", "Noseslide", "Tailslide", "Bluntslide", "Blunt", "Nosebluntslide", "Noseblunt", "Darkslide"],
};
```
```javascript jsonion/trieOfArrayUnique
/*/////
  creating routing index pointers to handle 
  keyword autocomplete queries more efficiently
 (keywords are disintegrated into per-char paths)
                                          */////
class TrieOfArrayUnique extends Object {
      #trie = new Object();
      #recurse; #terminated; #terminator;
  //////////////////////////////////////
  constructor (cfg={...}) {
        ////// cost of ensuring against override
        delete cfg.q;
        delete cfg.add;
        delete cfg.get;
        delete cfg.remove;
        delete cfg.formatInputs;
        delete cfg.recurse;

         super(cfg);
        ////// this property assigning fails

        Object.assign(this, cfg);
        ////// this works, though
  }

  formatInputs (keys,
                vals=null,
                  fn=null) {}
  recurse (keys, vals, fn) {}
   ////    generalist class factory function
   ////    ... of `jsonion/trieOfArrayUnique`
   ////    ... try TrieOfArrayUnique XML AST!
}
```

One interesting observation to make in above codeblock is that, when extending the `Object` class (`JSObjects.h` in Chromium), the new subclass can't use `super` to push input prop objects and configurations up into the `Object` superclass directly. Is this a little note to assume whether then any of these props would likely use the built-in hash table capacity?

> You should avoid calling any Object.prototype method directly from the instance, especially those that are not intended to be polymorphic (i.e. only its initial behavior makes sense and no descending object could override it in a meaningful way).
>
> -- <cite>[`Object` constructor at `developer.mozilla.org`][1]</cite>

Very likely, because of the way of accessing properties in collections in JS syntax, the above citation explains that `Object` properties are moved entirely out of the way for the one purpose of naming any custom paths. Static `Object` methods are exposed, instead.

In terms of memory allocation this implies that any getter and setter methods, applied in context of one `Object`, rely on hash table resolver's activity. In online documentation, however, it has been denoted that `JSObject` maintains the hash table index only upon reaching a certain size in entries count.

Before the next struct in example, what does this mean for the alphabet, is it too long already and each new entry requires hashing to run each level deeper?

```javascript
class keywordTrie extends TrieOfArrayUnique {
  parse (str, collection, idx=false) {
   var
  count = 0,
  words = str.split(this.rxSplit).forEach(str =>
          {
      ////  code needs tweaking ("To Fakie")
            if (!str.length) return;
            else count++;

             let term = str.toLowerCase();
                 
                 this.updateTrie(str, term,
                 collection, idx);
          });
          return words;
  }

    // the next method is suitable for strings,
   //  but for DOM tree AST we need static index
  //  (published `trieOfArrayUnique`
  //     becomes `TrieOfArrayUnique`, reactive)

  updateTrie (str, term, collection, idx) {
    this.recurse([...term], term, onLeafWordNode);
      //// clone [...term] string to char array
    function onLeafWordNode (wordNode, term) {
     return {
      //// create new trie leaf node
       term, n: 1,
        ref: {[collection]: [{idx, n: 1}]}
      //// where's "To Fakie" now
     }}
  }
}

var keywordTries
 = { TRICKS: new keywordTrie(),
      spots: new keywordTrie() };

for (let [key, arr] of Object.entries(TRICKS)) {
for (let [idx, str] of Object.entries(arr)   )
     keywordTries.TRICKS.parse(str, key, idx);
}

```

If considering memory usage, coming from this type of objects, it's important to note that there is maintaining of a list of consumers, variables out there which currently reference to any given JSObjects subclass instance, which are stored in memory in reverse-reference, too (along the way one can learn that objects as they are invoked stay in memory until this list turns empty). This probably means that these `Trie` objects are freed up by cascading down the keyword tree (can you imagine that, char by char, from root to suffix, kind of like with `Solitaire`, the card game).


Anyway, when checking for numerous string keys on `JSObject` at once (like using `object destructure syntax`)... Because surely key orders kept in memory don't equal across objects in transaction, the datasets and thereby applying schemas, the query for values in key-value pairs, even with all the hashing tables, it must be atleast somewhat less efficient than when getting `slice` of one `Array`.

In case of `Array` these take place inline in a sequence of cosequent indexes, and mapping data state to variables made possible via the more efficient array destructuring syntax). It must be akin to being on streak, yeah sure, like in `Sonic Dash` and `Subway Surf`!

Then the coding project to mitigate this, easy or difficult, it literally means that data after schema are supplied in bulks -- and perhaps this simply means a more talkative API with more access points, as these are routes where variables take stage as planned for proper usage and then structured for the various use-cases (perhaps with their own key-value store cache). Yes, and keyword index applied to resolve them. Hey, maybe a real chatbot can help fetch data the first time!

```javascript reactive.elementPrototype.js
import {getPropPaths} from './reactive.props.js';
//////  this method attaches to HTMLElement
//////  … it's looping the DOM for trie AST

 ///////////////////////////////////////////
// there's still time for custom tag class
class reactiveElement extends HTMLElement {
  static AST 
   = new Map(); /*  1. getPropPaths<arrayTrie>
                    ... requires
                       <template id={onion-key}>
                   (can receive AST from API)
                                              */
         AST; /* 2. evalPropPaths<flatAST>    /
                    ... after cloning tpl    /
        /////       clones `onion-key` attr /
                    props to render AST    /
                   `sortAdd`,`onion-key` */
        
  static AST;  props;  state; 
         AST = reactiveElement.AST
                              .get(this);
  constructor (props) {
  ///  … super(props) ... subclass call

  ///  time to register custom element
  ///  … `reactive.ctl` finds trie AST
    this.AST = container.AST ||
             = container.AST
             = this.enumPropPathscontainer.getPropPaths();
      /*   [!!!]   merge above and the below */
   var AST
   if (!this.AST
   &&   reactiveElement.AST.get(this))
   //   reactiveElement, this.getPropPaths

  ///  props to render where
  ///  comparing edited props and traversing AST 
  ///  … thus live AST nodes need:
  ///    sorting up in path groups
  var  omitProps = new Array();
  }

  // method's imported from `reactive.props.js` 
  getPropPaths;

  // browser API for `custom tag` elements
     connectedCallback() {}
  disconnectedCallback() {}
       adoptedCallback() {}

/*/////// or scripted in React legacy code
  static componentWillMount (container) {}

  static componentWillReceiveProps (instance,
                                    props) {}
 *///////
}

```

```javascript example-component
///  i.e. `state` is defined at a reactive hook
for (let inArrRow of DB[collection]
                  .sort(sortingKey)
                  .slice(page_idx * perPage, 
                         page_idx * perPage
                                  + perPage))
    {
let [userId, displayName, streakData] 
 =   inArrRow.slice(1)
///  inArrRow.0 is a pointer object (next lines)
///  inArrRow.get("userId", 
///                "displayName",
///                  "streakData");


///  the following code block is from automation
let  onionKey = String.fromCharCode(
this.onionKey ++ );

let [key] = this.AST.find(element => 
                          element.getAttribute(
                                  "onion-key")
                            ===    onionKey);

if (this.props.userId !== userId
||  this.displaythis.setState({   ///////////////////////////////////////////////////////
  user
})
///  this happens on `componentReceivedProps`
///   ... I attempt to create reactive tpl ctl
    }
```

Surely, code sprints and mindfulness are required to get through and schematize these in-array sequences, ready to expect query use-cases which require specific variables. Well is this the type of freedom one intends, to plan quality coded applications?!

Another point to practice internals, with outcome similar to the above, it is the potential implication within the scope of Chromium's V8 engine and its `hidden classes`, however minor in improving actual efficiency. There, the in-memory compiled JS code and its just-in-time cache are streamlined, thriving on equal order of appearance for variables as they are evoked. This holds true for any actual combination of variables to receive a value, as instead of not holding a value, besides any other database objects, destructured or  accessed in depth by some indexing key. 

With V8, like molds, when inputs occur in various combinations for each new combination of invoked variables, more new `hidden classes` are invoked in turn. Then, instead of assigning one or two variables more per each one execution of a given consumer method, keep in mind that these data objects already exist in memory after `JSON.parse`. The current cost these methods assert is then to create a new `hidden class` and this does indeed imply a processing cost slightly bigger (perhaps not only for the time-keepers). Array destructuring syntax in JS could help with that, lining up values in streak after any designate validation requirements,
such as in the case of inArrayTypes ...
- inArrayType.{transformerFnName}
- inArrayType.{boundaryRequirement}
- inArrayType.{validatorFnName}
... for the savvy, mentioned types are safely decoupled from its reference class inArrayType, aliasing `this` context within (thus improving syntax readability by allowing for lining up their alias names in schema definitions). I can't yet understand the significance myself!

This idea is in its real impact during the current web perhaps a small one, but it ties in some interesting developer challenges, as naturally entailing the current web standards. As the V8 engine is readily implemented by Node.js on server-side, this wild feature widely discussed hereby in this publication remains identical to developer experience on client-side, for the browser.

Thus, for the purposes mentioned hereby, one web application could share these utilities among the browser and the server-side API, for example while preparing to cache DB query results in-memory, according to the inArrayType schema that's internally coordinated. But hey, maybe its best to just use Elastic Search with Node.js, it really depends on access patterns for various data types!
... interestingly, author of a backend DB solution mentioned during a conversation with me how he uses this kind of root type structs himself -- why not just send them to a browser, kept undefined in private places?

Yes, currently this discussion seems quite theoretical. However, was `jsonion` made a Tor node this line of thought would lead to a seriously interesting implication. Because of sudden flexibility of database schemas out there in the wild, adopting standardized collection schemas to bridge the gap might in turn become the norm. There's one all-encompassing JSON-LD from Google's at schema.org, and then there's articles on Wikipedia which keep changing but maintain a static article name in URL. So there is objective, globally accessible reference points!

Not everyone running their Node.js is a keen frontend developer to implement all possible `key => alias` pairs in more generic service views. Structure-wise, these might share externally to dictate keymaps and aliases for schema keys across all the likely similar collections. Maybe the standardized collection schemas, or relevant Wikipedia article URLs as key, maybe these will then be used to translate data schema among sharing peer-to-peer nodes. After all, what we're talking about here is data validation where function input arguments are assigned parameters in preconceived sequences!

Shared among peers and imported in a running codebase, a defined schema is much needed to properly validate, before assimilating each document node to a fitting schema to run these various custom collections on the frontend. However inconceivable this may nowadays when the frontend doesn't flex as much, to fit various collections there on the shelf in an immersive experience!

Granted, there is evaluating checksums per document nodes for peer-to-peer validation with the checksum, as this requires maintaining a strict schema before it can reach relevancy, at least with blockchain validation fully in view. There is an obvious exception to consider, that the scope of checksum validation is hereby limited per each distinct document node, atleast for the day to day routine. Only per significant events is there possible an extension of one checksum to public publications per user while fitting various collections' schema. As filtered for presentation, broadcast at snapshot event. Meet the leap second!

The underlying layer sure enough doesn't mean public presentation for in-memory data. If there isn't static prop order found at JSObject's then a parsing entity could take charge right in place of `JSON.parse` and render one. It is already enough to get those schema keys in order!

But to continue, through inArrayType JSON datasets a special form similar to `entries` may be observed. It's the kind of object that's found resulting from the built-in global `Object.entries` method, commonly applied while traversing JSObjects. But listen, if there is a real need to automatically translate parts of schema by alias, is it more efficient when data is directly transformed to inArrayType per inbound stream socket package, i.e. via `ReadableStream`, originating from a DB API talking JSON:
[key => [...inArrayType /* definitions */)]]
... I'm not sure I know enough about cryptography to know if SSL layer will encrypt each packet, or is it that a whole JSON string is loading in buffer before decoding!

To wrap up, it should be noted somehow that this type of interface requires additionally the use of a several functional properties, supplied with the inArrayType object in chained context (value transformers, value boundary, and schema requirements are part of the codebase in disposition currently published). There's one interesting outcome, try think about writing schema in regular sentences using variable names (of transformers, requirements and validators)!

I was trigger-happy to publish my code disposition in jsonion.com/#airmoth repository, with inArrayType widely described in code. Hereby I claim this publication has been written after I have been working my way through quite a few of these coding tasks at the ground level, myself. My codebase for this challenge is rolled up quite long, but I dedicated extra attenion to make it suitable for mobile reading. Sometimes it takes one focused reader to slice up the codebase and throw each distinct module in its own designated script!

There I'm suggesting to maintain a per-collection database schema layer where DB entries, JSON document nodes preserve their key order in whole, as rooted in Array. Sure it would be nice to have a JSON global built-in object property just pile up schema key order data, right at the browser level!

Differencea in implementation include usage of ReaderStream and inArrayType on HTTP requests, and of course sometimes a layer to mimick seamless use like with JSObjects. But hold on, perhaps its better to just use and make use of Facebook's GraphQL and its schema syntax, spicing it up with some additional gql schema parsing!

This another case to consider, that GraphQL introduces writing `gql` statements on frontend, and because scripting modular schemas happens in string fragments the schema isn't lost. However, this approach will never be intended to provide a complete and ordered schema.

Under terms of efficiency and cross-platform compatibility there are some other interesting applications we shall explore later on, specifically about a C++ codebase to use with microcontrollers. Linking some of the C++ code for WebAssembly can lead to code reuse among platforms. There JSON objects are parsed from string as Uint8Array buffers and require making distinct build routes for C++ and JavaScript+WebAssembly ports in one codebase. This daunting task has historic implications for the LeapGest whitepaper, at least!

I was always pretty sure there isn't a native JSON.parse with custom method support at input to just hook and tap a few arrow functions right there. And I'm currently unsure whether GraphQL, addressing frontend data view requirements and clear schema, it in itself doesn't extend to BufferReader on some possibility, parsing JSON inbound traffic. Hm. Anyway, with its simplistic syntax it might just be enough ... that perhaps it could be utilized, extended, to finish this job with much less friction.

JSON may by historic reference link in a story from ages ago, from the cultural advent in ancient Greece, carry association with Jason who was yachting on his way up the Adriatic Sea back then. Is it just me, how is it that I live with my family right there on the course of this path... That's stunning.
