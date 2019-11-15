# Node.js best practices

(c) 2019 Jesús Leganés-Combarro 'piranna'

I've done several code auditories in the past to diferent companies, and found
some common errors and patterns that decrease code quality and increase their
technical debt. This is a guide and recopilation of best practices to try to
minimize and fix them.

This document was originally published at
https://github.com/piranna/Node.js-best-practices

**Disclaimer**: advices provided here are based on my own experience and could
be biased or opinionated. You don't need to follow all of them, but I've tried
to use common sense and being generic when writting them so could be a good
starting point for your own rules. It's mostly focused on Node.js development,
but the advices are generic enough to be applied to other platforms. In that
way, this document don't provide info about how to use the diferent tools, but
provides links to their docs so you can adjust them to your fits. If you think
something can be explain in a more generic way without loosing its essence, I'm
open for reviews and pull-requests :-)

This is not the only Node.js best practices guide you can found, if you want one
so big can blow your mind and transform you in a better developer (and maybe a
better person) take a look at https://github.com/goldbergyoni/nodebestpractices.

## Sponsors
- [Bpost](https://www.bpost.be)
- [Tata Consultancy Services](https://www.tcs.com)

## License

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Licencia de Creative Commons" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" href="http://purl.org/dc/dcmitype/Text" property="dct:title" rel="dct:type">Node.js best practices</span> by <a xmlns:cc="http://creativecommons.org/ns#" href="https://github.com/piranna" property="cc:attributionName" rel="cc:attributionURL">Jesús Leganés-Combarro 'piranna'</a> is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Reconocimiento-NoComercial-CompartirIgual 4.0 Internacional License</a>.
