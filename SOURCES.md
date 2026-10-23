# Sources, scope, and modernization decisions

This collection synthesizes guidance rather than reproducing the books chapter by chapter. It focuses on decisions an agent must make when reviewing, writing, refactoring, testing, or compiling Delphi. The four PDFs total **2,276 pages**. Their contents were extracted for navigation and selected technical sections were consulted; this is not a claim that every example was compiled or every assertion in every page was independently verified.

## Editions and page conventions

| ID | Supplied edition | PDF pages | Citation convention |
| --- | --- | ---: | --- |
| H | Marco Cantù, *Object Pascal Handbook — Delphi 11 Alexandria Edition*, November 2021 | 550 | Printed body page + 1 = PDF page |
| E | Marco Cantù and Paweł Głowacki, *Expert Delphi*, second edition, Packt, 2024; ISBN 9781805121107 | 424 | Numbered body page + 23 = PDF page |
| F | Alister Christie, *Code Faster in Delphi*, 2020 | 163 | Printed decimal page + 3 = PDF page; binary page decorations are not separate page numbers |
| M | Marco Cantù, *Mastering Delphi 5 — 2025 Annotated Edition*, release 0.2, February 17, 2025 | 1,139 | Use the PDF page, matching displayed body pagination in the consulted chapters |

Offsets apply to the numbered body, not covers or Roman-numbered front matter. These locators refer to the supplied editions, not other printings. The M title/copyright pages contain inconsistent historical/date wording; the 2025 preface explicitly identifies the base text as covering Delphi 5 released in 1999. Its footnotes, not a rewritten body, carry most modernization comments.

## Coverage map

| Source sections | Skills and treatment |
| --- | --- |
| H ch. 1–4 | Language, strings, build: units, types, initialization, routines, directives |
| H ch. 5–6 | Language, memory, collections, strings, I/O: arrays/records, managed records, UTF-16, encodings |
| H ch. 7–10 | Language, memory, VCL, components: objects, dispatch, exception safety, properties/events, RAD/OOP |
| H ch. 11–13 | Memory, interop, components: interfaces, class references/helpers, ownership and leak diagnosis |
| H ch. 14–15 | Collections, concurrency: generics, ownership-aware containers, captured variables |
| H ch. 16–18 | RTTI, components, I/O: metadata lifetime, TValue, attributes, streams, persistent/component base classes |
| H appendix/history/glossary | Context and terminology; not converted into prescriptive modern rules |
| E ch. 1–2 | Build, language: toolchains and modern language overview |
| E ch. 3–4 | I/O, concurrency: streams/JSON/XML, threads, locks, tasks/futures |
| E ch. 5–9 | FireMonkey, components: layouts, 2D/3D, styles, frames, services, lifecycle, desktop/mobile separation |
| E ch. 10–13 | Data, services: FireDAC, bindings, HTTP/REST, WebBroker, DataSnap, RAD Server |
| E ch. 14 | Build/testing/FireMonkey: deployment distinctions, source control, test framework use |
| E ch. 15 | Forward-looking context; no speculative product trends made mandatory |
| F language/RTL sections pp. 71–113 | Memory, collections, concurrency, testing, RTTI, data |
| F editor/designer sections pp. 16–57 | VCL, refactoring: navigation, generated code, coordinated form/source edits |
| F metaprogramming pp. 136–148 | Refactoring, data: repeatable migration, structural parsing, scoped reFind transformations |
| F tooling, shortcuts, physical environment, career sections | Environment-specific or human productivity advice; not imposed on an agent or used to require plugins |
| M ch. 1–3 | Language context; modern handbook takes precedence for current language semantics |
| M ch. 4–8 | VCL: component/form fundamentals, multi-form lifetime and resource behavior |
| M ch. 9–12 | Data: enduring dataset/field/state concepts; BDE/Paradox and outdated deployment recipes excluded from new-work recommendations |
| M ch. 13–16 | Components and interop: persistence, custom controls, packages, DLL/COM contracts with modern bitness checks |
| M ch. 17–19 | Concurrency, testing, interop: enduring concepts; legacy scheduling and obsolete translation tooling not recommended |
| M ch. 20–21 | Services: architectural context and explicit retirement of browser ActiveForms/CORBA recommendations |
| M ch. 22 | VCL: repaint-driven graphics and resource lifetime; FMX graphics use E instead |

Chapter ranges identify the conceptual foundation. Each topic's source footer provides narrower locators where available. Procedures for production failure handling, testing, security, deployment reporting, and agent behavior are explicitly marked editorial synthesis, not represented as quotations or direct author prescriptions.

## Concrete corrections and boundaries

| Evidence | Decision carried into the skills |
| --- | --- |
| M PDF 5–6, 2025 preface | Read original text together with annotations; absence of a footnote is not evidence of currency |
| M PDF 401 footnote 210; PDF 403 footnotes 213–217 | BDE no longer ships; preserve dataset concepts and favor appropriate current data libraries; retain ADO when the task calls for it |
| M PDF 184 footnote 144 | Do not preserve 32-bit pointer assumptions; Tag now uses a pointer-sized integer |
| M PDF 675 footnote 352 | Directly loaded binaries must match process bitness, including drivers and IDE packages |
| M PDF 804 footnote 364 | Treat old timer/message-based background examples as historical; choose actual workers for modern parallel work |
| M PDF 881 footnote 382 | Do not recommend the old Integrated Translation Environment as the supported default |
| M PDF 943 footnote 408 | Browser ActiveForms/ActiveX embedding is obsolete, distinct from COM's continuing Windows role |
| M PDF 1001/1007 footnotes 431, 443–444 | Do not resurrect retired MIDAS transport choices or Delphi CORBA support |
| H printed pp. 182–184 | Separate one-based normal string indexing from zero-based helper access; inspect directives |
| H printed pp. 129–133 | Dynamic arrays alias storage; do not assume string-like copy-on-write |
| H printed pp. 316–318 | Temporary interface references can invalidate a raw object reference |
| H printed pp. 375–377 | Destruction must tolerate partial construction; protect earlier acquisitions before later ones |
| H printed p. 411, abbreviated object-container discussion | Do not infer that ownership is unconfigurable; inspect target RTL constructors and flags |
| E printed p. 84 and Athens release documentation | JSON Data Binding Wizard is 12+, not an 11 requirement; generated runtime dependencies need their own check |
| E printed p. 382 and DUnitX primary examples/maintainer clarification | Per-test Setup/TearDown differs from per-fixture SetupFixture/TearDownFixture |
| E ch. 4 demonstrations | Add lifetime, cancellation, completion, and failure observation before treating short task examples as production patterns |
| F printed pp. 84–85 | Counter frequency describes resolution; avoid treating it as a guarantee of measurement accuracy |
| F printed pp. 136–145 | Component migration requires behavior/property/resource mapping, not blind text substitution |

## Primary documentation consulted

Checked 2026-10-23. Embarcadero content is versioned; a Florence page alone does not prove a feature existed in Alexandria. Some DocWiki pages intermittently returned HTTP 403. Search-index text was available for Custom Managed Records; the handbook independently supplies its baseline semantics. The TObjectList constructor page could not be retrieved, so the skills tell the agent to inspect the installed RTL instead of claiming a complete documentation audit.

- [RAD Studio Florence documentation](https://docwiki.embarcadero.com/RADStudio/Florence/en/Main_Page)
- [Compiler Versions](https://docwiki.embarcadero.com/RADStudio/Florence/en/Compiler_Versions): compiler identifiers only; product/folder numbers are not inferred from this table.
- [Florence changes](https://docwiki.embarcadero.com/RADStudio/Florence/en/What%27s_New): internal numbering and newer syntax.
- [Athens changes](https://docwiki.embarcadero.com/RADStudio/Athens/en/What%27s_New): JSON wizard version boundary.
- [Custom Managed Records](https://docwiki.embarcadero.com/RADStudio/Florence/en/Custom_Managed_Records)
- [Using Reference Counting](https://docwiki.embarcadero.com/RADStudio/Florence/en/Using_Reference_Counting)
- [TThread.Queue, Alexandria](https://docwiki.embarcadero.com/Libraries/Alexandria/en/System.Classes.TThread.Queue)
- [FireDAC multithreading, Alexandria](https://docwiki.embarcadero.com/RADStudio/Alexandria/en/Multithreading_%28FireDAC%29)
- [Managing Transactions, FireDAC](https://docwiki.embarcadero.com/RADStudio/Florence/en/Managing_Transactions_%28FireDAC%29)
- [MSBuild](https://docwiki.embarcadero.com/RADStudio/Florence/en/MSBuild_Overview)
- [Building a Project Using an MSBuild Command](https://docwiki.embarcadero.com/RADStudio/Florence/en/Building_a_Project_Using_an_MSBuild_Command)
- [Platform Status](https://docwiki.embarcadero.com/PlatformStatus/en/Main_Page): linked for current lookup, without copying a frozen SDK matrix.
- [DUnitX examples](https://github.com/VSoftTechnologies/DUnitX/blob/master/Examples/DUnitX.Examples.General.pas) and [maintainer's fixture-lifecycle clarification](https://github.com/VSoftTechnologies/DUnitX/issues/311).
- [OpenAI skill packaging](https://learn.chatgpt.com/docs/build-skills) and [Claude Code skills](https://code.claude.com/docs/en/skills): shared SKILL.md format and personal installation.
- [ETHSkills](https://ethskills.com/): consulted as a distribution model for a small entry point and remotely readable topics; no Ethereum content is incorporated.

## Source handling

The published collection contains original explanations, short original examples, and citations. It does not include the book PDFs, extracted text, images, or their example projects. Book rights remain with their respective authors/publishers. Selecting a license for the new collection is separate from citing those books.
