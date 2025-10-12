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