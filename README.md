# thrift-clj

Using Thrift from Clojure as if it was Clojure.

[![Build Status](https://travis-ci.org/xsc/thrift-clj.svg?branch=master)](https://travis-ci.org/xsc/thrift-clj)
[![endorse](https://api.coderwall.com/xsc/endorsecount.png)](https://coderwall.com/xsc)

## Usage

__Leiningen ([via Clojars](http://clojars.org/thrift-clj))__

[![Clojars Project](http://clojars.org/thrift-clj/latest-version.svg)](http://clojars.org/thrift-clj)

Make sure to additionally include a [slf4j](http://www.slf4j.org/)-compatible logger - e.g.
[logback](http://logback.qos.ch/) via:

```clojure
[ch.qos.logback/logback-classic "1.0.13"]
```

__Note__: Tested with the Thrift 0.9.0 compiler. Since this depends massively on the generated code, make sure to use
that version (or any other one that was tested with this library).

__Automatic Thrift Compilation__

I recommend [lein-thriftc](https://github.com/xsc/lein-thriftc) for automatic compilation of Thrift IDL files to Java
class files.

## Example

A working example demonstrating Service and Client implementation should always be available as
[thrift-clj-example](https://github.com/xsc/thrift-clj-example). A small peek follows.

### Accessing Types

__Thrift__

```thrift
namespace java org.example

struct Person {
  1: optional string firstName,
  2: string lastName,
  3: byte age
}
```

Compile to Java using Thrift and add to Leiningen's classpath. (see `:java-source-paths`)

__Clojure__

```clojure
(require '[thrift-clj.core :as thrift])
(thrift/import
  (:types [org.example Person]))

(def clj-p (Person. "Some" "One" 99))
;; => #ns_1071852349.Person{:firstName "Some", :lastName "One", :age 99}

(def thr-p (thrift/->thrift clj-p))
;; => #<Person Person(firstName:Some, lastName:One, age:99)>

(class clj-p) ;; => ns_1071852349.Person
(class thr-p) ;; => org.example.Person
```

### Implementing a Service

__Thrift__

```thrift
namespace java org.example

// ... 'Person' struct from above ...

service PersonIndex {
    bool storePerson(1:i32 id, 2:Person p),
    Person getPerson(1:i32 id)
}
```

__Clojure__

```clojure
(require '[thrift-clj.core :as thrift])
(thrift/import
  (:types [org.example Person])
  (:services org.example.PersonIndex))

(defonce person-db (atom {}))
(thrift/defservice person-index-service
  PersonIndex
  (storePerson [id p]
    (boolean
      (when-not (@person-db id)
        (info "Storing Person:" p)
        (swap! person-db assoc id p)
        true)))
  (getPerson [id]
    (info "Retrieving Person for ID:" id)
    (@person-db id)))

(thrift/serve-and-block!
  (thrift/multi-threaded-server
    person-index-service 7007
    :bind "localhost"
    :protocol :compact))
```

### Running a Client

```clojure
(require '[thrift-clj.core :as thrift])
(thrift/import
  (:types [org.example Person])
  (:clients org.example.PersonIndex))

(with-open [c (thrift/connect! PersonIndex ["localhost" 7007])]
  (PersonIndex/storePerson c 1 (Person. "Some" "One" 99))
  (PersonIndex/getPerson c 1))
```

## Tests

You can run [Midje](https://github.com/marick/Midje) tests using the following Leiningen command:

```
lein midje-all
```

Make sure that the Apache Thrift compiler is installed.

## Roadmap

- asynchronous client/server
- union?
- exceptions
- tests & documentation
- ...

## Related Work/Inspiration

- [Apache Thrift](https://github.com/apache/thrift)
- [Plaid Penguin](https://github.com/ithayer/plaid-penguin)
- [lein-thriftc](https://github.com/xsc/lein-thriftc)
- [thrift-clj-example](https://github.com/xsc/thrift-clj-example)

## License

```
The MIT License (MIT)

Copyright (c) 2013-2015 Yannick Scherer

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
