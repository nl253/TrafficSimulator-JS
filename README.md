# Traffic Simulator 

- generate mock traffic
- for now just GET requests

## API

### Basics

```js
const { TrafficSimulator, SECOND } = require('traffic-simulator');
const graph = { /* see below */ };

const opts = {
  delayRate:    1 * SECOND,
  minDepth:              2,
  maxDepth:             20,
  minTmOnPage:  1 * SECOND,
  maxTmOnPage: 30 * SECOND,
  nClients:             50,
  doLog:              true,
};

const ts = new TrafficSimulator(graph, opts);
ts.simulate();
```

### Default Options

```javascript
const MILLISEC = 1;
const SECOND   = 1000 * MILLISEC;

const DEFAULTS = {
  nClients:              10,
  delayRate:     1 * SECOND, // time between clients are spawned
  minTmOnPage:   3 * SECOND,
  maxTmOnPage: 120 * SECOND,
  maxDepth:              20, // max #clicks that each client makes
  minDepth:               2, 
  doLog:               true,
}
```

### Overridable Methods (via Inheritance)

```typescript
interface Printable { toString(): string; }

protected get randURL():    string; // start URL when creating a new client worker
protected get randDepth():  number; // length of new client worker tour
protected get randTmOnPg(): number; // time on the current page after each transition

protected nameFunct(idx: number): string; // name for new client workers
protected warn(msg: Printable):   void;   // defaults to console.warn
protected log(msg: Printable):    void;   // defaults to console.log

// You will have access to all opts: this.nClients, this.delayRate ...
```

### EventEmitter API

The following events are emitted along with the data specified in parenthesis.

- randURL(url: string)
- depth(workerName: string)
- exit(workerName: string)
- null(workerName: string)
- spent(workerName: string, url: string, timeSpent: number)
- spawn({ workerName: string, url: string, depth: number }, atTime: Date)

### Link Graph

For a complete example see `examples/simple.ts`.

```javascript
// adjacency list (floats are transition PROBABILITIES, they MUST add up to 1.0)
// 
// stackoverflow => google (p = .2)
// stackoverflow => exit (p = .8)
// news.ycombinator => news.ycombinator => .2
// news.ycombinator => exit (p = .8)
// 
// etc.
const EXAMPLE_GRAPH = {
  'https://stackoverflow.com/questions/951021/what-is-the-javascript-version-of-sleep': {
    'https://www.google.co.uk/search?newwindow=1&source=hp&ei=cVDHXPOtF5HosAfKnJrgDw&q=javascript+sleep+await&oq=jav&gs_l=psy-ab.1.0.35i39l2j0i20i263j0j0i131j0j0i20i263j0i131j0j0i131.889.1455..2407...0.0..0.131.347.3j1......0....1..gws-wiz.....0.8oIEbZdX7Es': 0.2,
    '%exit%': 0.8,
  },
  'https://news.ycombinator.com/': {
    'https://news.ycombinator.com/news?p=2': 0.2,
    '%exit%': 0.8,
  },
  'https://news.ycombinator.com/news?p=2': {
    'https://news.ycombinator.com/news?p=3': 0.1,
    '%exit%': 0.9,
  },
  'https://www.google.co.uk/search?newwindow=1&source=hp&ei=cVDHXPOtF5HosAfKnJrgDw&q=javascript+sleep+await&oq=jav&gs_l=psy-ab.1.0.35i39l2j0i20i263j0j0i131j0j0i20i263j0i131j0j0i131.889.1455..2407...0.0..0.131.347.3j1......0....1..gws-wiz.....0.8oIEbZdX7Es': {
    'https://stackoverflow.com/questions/951021/what-is-the-javascript-version-of-sleep': 0.8,
    'https://flaviocopes.com/javascript-sleep/': 0.2,
  },
  'https://github.com/': {
    'https://github.com/': 0.4,
    'https://github.com/search?utf8=%E2%9C%93&q=node&type=': 0.5,
    '%exit%': 0.1,
  },
  'https://www.w3schools.com/jsref/met_win_settimeout.asp': {
    'https://www.w3schools.com/jsref/prop_win_opener.asp': 0.1,
    'https://www.w3schools.com/jsref/prop_win_sessionstorage.asp': 0.2,
    '%exit%': 0.7,
  },
  'https://github.com/search?utf8=%E2%9C%93&q=node&type=': {
    'https://github.com/nodejs/node': 0.6,
    '%exit%': 0.4,
  },
};
```
