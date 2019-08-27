
[MarkLike (v0.1)](HTTPS://GitHub.com/LongTengDao/MarkLike/)
=================

MarkLike uses the way as simple and intuitive as possible, to provide an as general and extensible as possible high descriptive profile writing form (in addition to article writing). It referred to the syntax of XML and TypeScript to some extent.

MarkLike entrusted the responsibility of whether the final document was intuitive or not to the writer's ability to adapt to local conditions. After [MarkDoc](HTTPS://GitHub.com/LongTengDao/MarkDoc/), [TOML](HTTPS://GitHub.com/toml-lang/toml/) has solved the problems in their respective fields in the best way, another practice of grammatical orientation is used to satisfy the remaining usage scenarios.

---

There are only three original atomic values in MarkLike: existence, integer and text.

There is only one original data structure, which allows key-value pairs to coexist with index values and keeps them in order.

```
<x: a: b:=1 c:=-1 d:="inline string" e:=""">
	multi-line string
<y:=""">
	multi-line string
<z: a: b:>
	<c:="inline string">
	<d: :="inline string">
		<: i: j: :>
		<:="inline string">
		<:>
		<m:>
```

The colon is preceded by a key name, and leaves empty means adding an index which similar to array item.

This example can be implemented in JavaScript as follows:

```js
$(
	["x", $(
		["a"],
		["b", 1n],
		["c", -1n],
		["d", "inline string"],
		["e", "multi-line string"],
	)],
	["y", "multi-line string"],
	["z", $(
		["a"],
		["b"],
		["c", "inline string"],
		["d", $(
			[0, "inline string"],
			[1, $( ["i"], ["j"], [0] )],
			[2, "inline string"],
			[3, $()],
			["m", $()],
		)],
	)],
)
```

The `$` can be understood expediently as follows:

```js
function $ (...entries) {
	class Structure extends Map {
		get length () { return [ ...this.keys() ].filter(key => typeof key==='number').length; }
		get names () { return [ ...this.keys() ].filter(key => typeof key==='string'); }
	}
	return new Structure(entries);
}
```

---

The colon is followed by a custom type constructor, similar to "tag" in YAML. Therefore:

```
<key:TypeA subKey:TypeB>
```

Correspondingly, it can be understood as the following implementations in JavaScript:

```js
const root = $(
	["key", $(
		["subKey"]
	)]
);
TypeB(root.get("key"), "subKey", root.get("key").get("subKey"));
TypeA(root, "key", root.get("key"));
```

Custom constructors are executed from the inside out. In addition to the transforming values, implementation should also allow constructors to know the key name, modify the key name, and even access the mounted target data structure, in order to achieve the feature similar to decorator.

The empty case after the colon in the previous article indicates that the original parsing results are mounted directly.

---

Bare key names and custom types only accept more than one `a`~`z`, `A`~`Z`, `_`, `-`, `.` characters, otherwise they should be written as strings.

Whether `:` can be omitted when key name and custom type annotation exist only one, and whether literal quantities represent key name or custom type in this case, the specification has not yet been determined.
In addition to simplifying common usage forms as much as possible, it should also be considered to achieve formal uniformity with the following string escape formats.

---

The escape character in the string is `<`. ` <U+Hex>` represents an Unicode character, otherwise parsed by a custom character constructor. Such as:

```
<key="<U+61><quot><lt><br><emoji>">
```

It will be implemented as:

```js
$(
	["key", String.fromCodePoint(0x61)+$$('quot')+$$('lt')+$$('br')+$$('emoji')],
)
```

If a string containing character escalation is located on a key with a custom type, the implementation should allow the string to be received in an interpolation template structure:

```
<key:type="a<b>c">
```

Consideration should be given to achieving:

```js
const root = $(
	["key", ["a", $$("b"), "c"]],
);
type(root, "key", root.get("key"));
```

---

MarkLike files must be encoded as UTF and clearly can be specified by BOM or the first ASCII character; only CRLF or LF is understood as a newline character and the full text must be unified (and used as a concatenated character of multi-line strings); only Tab indentation is accepted, and empty lines and the indentation of empty lines are sensitive.
