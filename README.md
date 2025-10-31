# DBLP Scrape
## Install
```
pip install dblpy-lib
```

## DBLP Query Syntax
### Usage
```
case-insensitive prefix search: default
e.g., sig matches "SIGIR" as well as "signal"

exact word search: append dollar sign ($) to word
e.g., graph$ matches "graph", but not "graphics"

boolean and: separate words by space
e.g., codd model

boolean or: connect words by pipe symbol (|)
e.g., graph|network
```
- https://dblp.org/search?q=
- https://dblp.org/faq/1474589.html

### Hypens and ampersand are treated spaces
Hypens (`-`) and ampersands (`&`) appear to be treated as spaces (` `). E.g., `multi-agent` and `multi agent` return the same results:
- https://dblp.org/search?q=multi%20agent
- https://dblp.org/search?q=multi-agent

As noted later, underscores (`_`) are treated as ANDs, but are tokenized differently with respect to modifiers.

### ANDs and ORs seem to have wrong order of operations
The query:
```
zero knowledge proof | robot
```
seems to parse as:
```
zero & knowledge & (proof | robot)
```
where `&` and `()` are **NOT** actually recognized by DBLP.
- https://dblp.org/search?q=zero%20knowledge%20proof%20%7C%20robot

### Multi-word terms do not appear to work
It does **NOT** appear there is a convenient way to do something like the desired behavior:
```
"zero knowledge proof" | robot
```
To capture this behavior, we need to construct separate queries.
- https://dblp.org/search?q=zero%20knowledge%20proof
- https://dblp.org/search?q=robot

### Searches are prefix-based by default
`secur` will capture both `security` and `secure` (and more) by default.

### Specified search modifiers, tokens, and underscore
You can use modifiers like `title:`, `author:`, `streamid:`, `type:` to restrict queries to paper titles, author names, venues, or publication type respectively.

Modifiers seem to apply to the next token until a space (` `) is encountered. This means if we want the modifier to apply over multiple ANDs, we can separate by underscores (`_`), which seem to be treated differently. Don't put a space immediately after a modifier (i.e., `title:robot` is good, `title: robot` is bad).

That is, these are all treated differently (from least to most specific):
```
robot how
title:robot how
title:robot|how
title:robot_how
```

### Useful Modifiers

#### Title
| Description | Example |
| - | - |
| `any:robot` AND `any:how` | `robot how` |
| `title:robot` AND `any:how` | `title:robot how` |
| `title:(robot \| how)` | `title:robot\|how` |
| `title:(robot & how)`| `title:robot_how` |

where the left column is pseudo-code. I.e., `any:` implies the word can show up anywhere in the data (e.g., title, author, venue, etc.).

#### Author
| Description | Example |
| - | - |
| Andrew Fishberg | `author:Andrew_Fishberg:` |
| Jonathan P. How | `author:Jonathan_P._How:` |

where trailing `:` seem to be used to denote enums.

#### Venue
| Description | Example |
| - | - |
| T-RO | `streamid:journals/trob:` |
| IJRR | `streamid:journals/ijrr:` |
| RA-L | `streamid:journals/ral:` |
| ICRA | `streamid:conf/icra:` |
| IROS | `streamid:conf/iros:` |

#### Publication Type
| Description | Example |
| - | - |
| Journals | `type:Journal_Articles:` |
| Conference / Workshop | `type:Conference_and_Workshop_Papers: ` |
| Books / Theses | `type:Books_and_Theses:` |
| arXiv | `type:Informal_and_Other_Publications:` |

#### Publication Year
| Description | Example |
| - | - |
| year | `year:2025:` |

There does not seem to be a way to do multiple years.

## Zotero Translation Server
We can fetch abstracts using [Zotero's Translation Server](https://github.com/zotero/translation-server), which has support for a wide range of references (see full list in [translators repo](https://github.com/zotero/translators/)).

```
# git
cd ~/ws
git clone --recurse-submodules https://github.com/zotero/translation-server
cd translation-server

# docker
docker build -t translation-server .
docker run -ti -p 1969:1969 --rm translation-server

# usage
curl -d 'https://ieeexplore.ieee.org/document/10654550' -H 'Content-Type: text/plain' http://127.0.0.1:1969/web
```

### Sample Results
```
$ curl -d 'https://ieeexplore.ieee.org/document/10654550' -H 'Content-Type: text/plain' http://127.0.0.1:1969/web
[{"key":"9I9G2HHR","version":0,"itemType":"journalArticle","creators":[{"firstName":"Andrew","lastName":"Fishberg","creatorType":"author"},{"firstName":"Brian J.","lastName":"Quiter","creatorType":"author"},{"firstName":"Jonathan P.","lastName":"How","creatorType":"author"}],"tags":[{"tag":"Distance measurement","type":1},{"tag":"Three-dimensional displays","type":1},{"tag":"Robot sensing systems","type":1},{"tag":"Noise measurement","type":1},{"tag":"Antenna measurements","type":1},{"tag":"Measurement uncertainty","type":1},{"tag":"Noise","type":1},{"tag":"Distributed robot systems","type":1},{"tag":"multi-robot systems","type":1},{"tag":"range sensing","type":1},{"tag":"robotics in under-resourced settings","type":1},{"tag":"swarm robotics","type":1}],"publicationTitle":"IEEE Robotics and Automation Letters","title":"MURP: Multi-Agent Ultra-Wideband Relative Pose Estimation With Constrained Communications in 3D Environments","volume":"9","issue":"11","pages":"10612-10619","abstractNote":"Inter-agent relative localization is critical for many multi-robot systems operating in the absence of external positioning infrastructure or prior environmental knowledge. We propose a novel inter-agent relative 3D pose estimation system where each participating agent is equipped with several ultra-wideband (UWB) ranging tags. Prior work typically supplements noisy UWB range measurements with additional continuously transmitted data (e.g., odometry) leading to potential scaling issues with increased team size and/or decreased communication network capability. By equipping each agent with multiple UWB antennas, our approach addresses these concerns by using only locally collected UWB range measurements, a priori state constraints, and event-based detections of when said constraints are violated. The addition of our learned mean ranging bias correction improves our approach by an additional 19% positional error, and gives us an overall experimental mean absolute position and heading errors of 0.24 m and 9.5\\bm ^\\circ respectively. When compared to other state-of-the-art approaches, our work demonstrates improved performance over similar systems, while remaining competitive with methods that have significantly higher communication costs.","DOI":"10.1109/LRA.2024.3451393","ISSN":"2377-3766","date":"2024-11","url":"https://ieeexplore.ieee.org/document/10654550","libraryCatalog":"IEEE Xplore","accessDate":"2025-10-31T14:37:57Z","shortTitle":"MURP"}]
```

```
$ curl -d 10.1109/LRA.2024.3451393 -H 'Content-Type: text/plain' http://127.0.0.1:1969/search
[{"key":"ERG22USX","version":0,"itemType":"journalArticle","creators":[{"firstName":"Andrew","lastName":"Fishberg","creatorType":"author"},{"firstName":"Brian J.","lastName":"Quiter","creatorType":"author"},{"firstName":"Jonathan P.","lastName":"How","creatorType":"author"}],"tags":[],"publicationTitle":"IEEE Robotics and Automation Letters","journalAbbreviation":"IEEE Robot. Autom. Lett.","volume":"9","issue":"11","ISSN":"2377-3766, 2377-3774","date":"11/2024","pages":"10612-10619","DOI":"10.1109/LRA.2024.3451393","rights":"https://creativecommons.org/licenses/by/4.0/legalcode","url":"https://ieeexplore.ieee.org/document/10654550/","title":"MURP: Multi-Agent Ultra-Wideband Relative Pose Estimation With Constrained Communications in 3D Environments","libraryCatalog":"DOI.org (Crossref)","accessDate":"2025-10-31T15:03:11Z","shortTitle":"MURP"}]
```