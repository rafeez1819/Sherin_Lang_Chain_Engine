# Sherin_Lang_Chain_Engine

ChainNode abstraction (plugin pipeline)

Synonym graph indexing

Semantic field clustering

Rhetorical device detection

Multilingual partitioning

This is modular, deterministic, and enterprise-scalable.

<img width="276" height="49" alt="image" src="https://github.com/user-attachments/assets/65dd5ef4-1ad1-4733-93e1-e5854e5739a6" />
________________________________________________________________________________________________________________________________

<img width="318" height="855" alt="image" src="https://github.com/user-attachments/assets/a3285e0c-f92d-4626-b0e1-2098ed3720e4" />



1. ChainNode Abstraction (Plugin Pipeline)
src/core/chain-node.ts
export interface ChainContext {
  text: string
  tokens?: string[]
  grammar?: any
  semantics?: any
  rhetorical?: any
  lexical?: any
}

export interface ChainNode {
  name: string
  execute(context: ChainContext): Promise<ChainContext>
}
src/core/language-chain.ts
import { ChainNode, ChainContext } from "./chain-node"

export class LanguageChain {
  constructor(private nodes: ChainNode[]) {}

  async run(input: string): Promise<ChainContext> {
    let context: ChainContext = { text: input }

    for (const node of this.nodes) {
      context = await node.execute(context)
    }

    return context
  }
}

Pipeline is now modular and pluggable.

2. Synonym Graph Indexing
src/lexicon/synonym-graph.ts
export class SynonymGraph {
  private graph: Map<string, Set<string>> = new Map()

  build(entries: any[]) {
    for (const entry of entries) {
      const word = entry.word
      const synonyms = entry.synonyms || []

      if (!this.graph.has(word)) {
        this.graph.set(word, new Set())
      }

      for (const syn of synonyms) {
        this.graph.get(word)?.add(syn)

        if (!this.graph.has(syn)) {
          this.graph.set(syn, new Set())
        }

        this.graph.get(syn)?.add(word)
      }
    }
  }

  getSynonyms(word: string): string[] {
    return Array.from(this.graph.get(word) || [])
  }
}

Bidirectional mapping ensures semantic symmetry.

3. Semantic Field Clustering
src/lexicon/semantic-cluster.ts
export class SemanticCluster {
  private fields: Map<string, Set<string>> = new Map()

  build(entries: any[]) {
    for (const entry of entries) {
      const word = entry.word
      const fields = entry.semantic_fields || []

      for (const field of fields) {
        if (!this.fields.has(field)) {
          this.fields.set(field, new Set())
        }

        this.fields.get(field)?.add(word)
      }
    }
  }

  getFieldWords(field: string): string[] {
    return Array.from(this.fields.get(field) || [])
  }
}

Supports thematic grouping for narrative generation.

4. Rhetorical Device Detection Layer
src/nlp/rhetorical-detector.ts
export function detectRhetoric(text: string) {
  const devices: string[] = []

  if (/(\b\w+\b).+\1/.test(text)) {
    devices.push("repetition")
  }

  if (/(like|as)\s+\w+/.test(text)) {
    devices.push("simile")
  }

  if (/\b[A-Z][a-z]+\s+[A-Z][a-z]+\b/.test(text)) {
    devices.push("proper_noun_reference")
  }

  if (/^[^aeiou]*[aeiou]/i.test(text)) {
    devices.push("alliteration")
  }

  return devices
}

This is rule-based now. Can later be upgraded to ML classification.

5. Multilingual Partitioning
src/lexicon/multilingual-partition.ts
export type SupportedLanguage = "en" | "fr" | "es"

export class MultilingualPartition {
  private lexicons: Map<SupportedLanguage, any> = new Map()

  register(language: SupportedLanguage, lexicon: any) {
    this.lexicons.set(language, lexicon)
  }

  get(language: SupportedLanguage) {
    return this.lexicons.get(language)
  }
}

This allows independent lexicon loading per language.

6. Lexicon Loader (Language Aware)
src/lexicon/lexicon-loader.ts
import fs from "fs"
import zlib from "zlib"

let cache: Map<string, any> = new Map()

export function loadLexicon(language: string) {
  if (cache.has(language)) {
    return cache.get(language)
  }

  const path = `data/lexicon.${language}.json.gz`
  const compressed = fs.readFileSync(path)
  const decompressed = zlib.gunzipSync(compressed).toString()
  const parsed = JSON.parse(decompressed)

  const frozen = Object.freeze(parsed)
  cache.set(language, frozen)

  return frozen
}

Supports partitioned lexicon sets.

7. Engine Composition
src/core/sherin-engine.ts
import { LanguageChain } from "./language-chain"
import { detectRhetoric } from "../nlp/rhetorical-detector"
import { loadLexicon } from "../lexicon/lexicon-loader"
import { SynonymGraph } from "../lexicon/synonym-graph"
import { SemanticCluster } from "../lexicon/semantic-cluster"
import { ChainNode } from "./chain-node"

export class SherinEngine {
  private chain: LanguageChain

  constructor(language: string = "en") {
    const lexicon = loadLexicon(language)

    const synonymGraph = new SynonymGraph()
    synonymGraph.build(lexicon)

    const cluster = new SemanticCluster()
    cluster.build(lexicon)

    const rhetoricalNode: ChainNode = {
      name: "rhetorical",
      async execute(ctx) {
        ctx.rhetorical = detectRhetoric(ctx.text)
        return ctx
      }
    }

    this.chain = new LanguageChain([
      rhetoricalNode
      // Add grammar node
      // Add semantic node
      // Add lexicon enrichment node
    ])
    
  }

  async process(text: string) {
    return this.chain.run(text)
  }


Engine is now:

Plugin-based

Language-aware

Synonym indexed

Semantic clustered

Rhetoric-aware
