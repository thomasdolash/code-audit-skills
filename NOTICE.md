# Third-party notices

This set is licensed under MIT (see `LICENSE`). Four of its skills adapt material from
MIT-licensed projects, and those notices must accompany any redistribution of the adapted
text. Each adapted skill also carries its own `## Attribution` section, so the credit
travels when a single skill directory is distributed on its own.

| upstream project | license | adapted by | what was adapted |
| --- | --- | --- | --- |
| [dzhng/jevgrep](https://github.com/dzhng/jevgrep) | MIT | `find-state-machines`, `find-overengineering` | the `find-*` naming, the class and core requirement of each, and one description near-verbatim |
| [JoseAntonioNuevo/test-suite-doctor](https://github.com/JoseAntonioNuevo/test-suite-doctor) | MIT | `find-verification-adequacy` | the slop-pattern catalogue and the per-case cost ranking |
| [petrkindlmann/qa-skills](https://github.com/petrkindlmann/qa-skills) | MIT | `find-redundant-checks` | the corpus-level audit, the coverage-equality rule and the destructive-safety ladder |
| [svyatov/supermatt](https://github.com/svyatov/supermatt) | MIT | `find-redundant-checks` | the move set and the guardrails |

## MIT license text, with the copyright holders named above

```
MIT License

Copyright (c) 2026 David Zhang                (dzhng/jevgrep)
Copyright (c) 2026 Matt Pocock                (svyatov/supermatt)
Copyright (c) 2026 Leonid Svyatov             (svyatov/supermatt)
Copyright (c) 2026 Petr Kindlmann             (petrkindlmann/qa-skills)
Copyright (c) 2026 Jose Antonio Nuevo         (JoseAntonioNuevo/test-suite-doctor)

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

## Other sources referenced

`find-duplicate-code` takes its class from the `duplicate-code` check in
[agent-sh/deslop](https://github.com/agent-sh/deslop), which reports added code of sixty
tokens or more that already exists elsewhere. That repository publishes no license file and
states no terms, so no text of its own is reproduced here; the check it names is the idea
credited, in the same way as the guidance below.

`find-complex-functions`, `find-oversized-units` and `find-verification-adequacy` take size
and complexity anchors from widely published review guidance: Linus Torvalds' note on
function length, the Never Nesting school, and ESLint's `max-lines-per-function` default of
50. None of that is licensed text, and none of it is reproduced verbatim.

No code from any listed project is included. These are prose methods; nothing here is
compiled or executed.
