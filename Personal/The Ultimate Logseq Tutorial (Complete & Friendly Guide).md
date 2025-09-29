# 🧩 The Ultimate Logseq Tutorial (Complete & Friendly Guide)

Welcome aboard! Logseq is a **privacy-first, open-source, local-first knowledge management system** — think of it as a digital brain that helps you organize tasks, knowledge, and ideas through *outlining, backlinks, and graph views*.  
It’s like a bullet journal, personal wiki, and second brain had a baby, and it speaks Markdown (and Org-mode if you prefer).

---

## 🌱 1. What is Logseq & Why Use It?
- **Outliner first**: Everything is a *bullet point*, and each bullet can be linked, tagged, or nested.  
- **Bidirectional linking**: If Page A links Page B, Logseq creates a backlink on Page B automatically.  
- **Personal knowledge graph**: Visual graph showing how your notes interconnect.  
- **Privacy-first**: Your notes are plain text (`.md` or `.org`) stored locally/synced via your method (Git, iCloud, Google Drive, Syncthing).  
- **Journaling & task management**: Automatically creates a daily journal page to capture thoughts.

---

## 💻 2. Installation
1. Download from [https://logseq.com](https://logseq.com)  
2. Available for **Windows, macOS, Linux**, and **Mobile (iOS/Android)**.  
3. Open it → choose a **graph folder** (this is just a folder on your computer that will hold your `.md` files).  
4. Boom, you have a graph! 🎉

---

## 🗂 3. Basic Structure
- **Graph:** A collection of pages (aka your whole Logseq workspace).  
- **Pages:** Each `.md` file in your graph. Created automatically when you type `[[Page Name]]`.  
- **Blocks:** Each bullet point. Can be moved, indented, linked, or embedded.  

```markdown
- This is a block
  - This is a sub-block
    - This goes deeper
- Another block on the same page
```

Blocks are *atomic units of thought* in Logseq.

---

## 🔗 4. Linking Notes
Two main ways:
1. **Page links** `[[Like this]]`  
2. **Tags** `#like-this`  

```markdown
- Meeting notes with [[Project A]]
- Ideas tagged with #brainstorm
```

- Pages get created automatically if they don’t exist yet.  
- Backlinks appear in the linked page’s “Linked References” section.  

---

## 📝 5. Journals
Logseq automatically creates a **Journal page per day** (`YYYY_MM_DD.md`).  
Perfect for:
- Daily notes
- Quick capture
- Adding tasks/events

```markdown
- [[2024-06-15]]
  - Research Logseq features
  - Call with #TeamX
  - TODO finish federation test
```

---

## ✅ 6. Tasks & Todos
Logseq has **built-in task management** with simple keywords:  
- `TODO` → task not started  
- `DOING` → in-progress  
- `DONE` → finished  
- `LATER` → postponed  
- `WAITING` → blocked  

```markdown
- TODO Write Logseq overview page
- DOING Federation test with B2C tenants
- WAITING Response from #AzureSupport
- DONE Finish internal doc
```

You can **query tasks** across your graph (next section 👇).

---

## 🔍 7. Queries (The Superpower)
Logseq supports Datomic-like queries, but here’s the friendly syntax first:

### Simple Queries
```clojure
#+BEGIN_QUERY
{:title "My Tasks"
 :query [:find (pull ?b [*])
         :where
         [?b :block/marker ?marker]
         [(contains? #{"TODO" "DOING"} ?marker)]]
}
#+END_QUERY
```

The above pulls all blocks with `TODO` or `DOING`.  

### Query Builders
For beginners — use **built-in query builder** (`/query`) that helps filter by tags, TODO states, properties.  

---

## 🏗 8. Properties & Metadata
Blocks can have properties, like YAML frontmatter but inline.  

```markdown
- Book:: "Thinking Fast and Slow"
- Author:: Daniel Kahneman
- Status:: #reading
```

These properties are queryable, meaning you can build a personal knowledge database.

---

## 🪄 9. Embedding & Transclusion
You can **embed blocks/pages inside other pages**.

### Embed a page
```markdown
{{embed [[Project A]]}}
```

### Embed a block
- Hover a block → click `Copy block ref`.  
- Paste → `((block-id))` appears.  

This way, you avoid duplication and maintain **one source of truth**.

---

## 🎨 10. Themes & Customization
- Logseq supports **themes via plugins** (Marketplace built-in).
- Popular plugins:
  - **Logseq Awesome Content** (templates & examples)
  - **Whiteboard** (visual graph)  
- You can also tweak `custom.css` inside your graph folder.

---

## 🤝 11. Syncing & Collaboration
- Logseq doesn’t impose sync → **you choose your sync strategy**: Git, Dropbox, iCloud, Syncthing.  
- There is a **Logseq Sync (paid)** service if you want end-to-end encrypted sync out of the box.  
- Collaboration: not real-time multi-user like Notion, but you can **share a synced folder via Git** for distributed collaboration.

---

## ⚡️ 12. Advanced Features
- **Whiteboards:** Create concept maps mixing blocks and drawings.  
- **Plugins:** Extend functionality with calendars, spaced repetition, Pomodoro timers.  
- **Publishing:** Publish your graph online (static site export).  
- **Block-level references:** Every bullet is addressable and reusable.  
- **Org-mode support:** If you’re an Emacs fan, you can switch to `.org` file format.

---

## 🏆 13. Best Practices
1. Don’t overthink: just start with daily journals.  
2. Use `[[links]]` liberally — don’t worry about structure upfront.  
3. Gradually refactor pages as themes emerge.  
4. Query tasks, don’t create separate todo apps.  
5. Use properties for structured data (Books, Contacts, etc.).  
6. Embrace that your notes will evolve into a **network**, not a rigid folder system.

---

## 🎉 Final Thought
Logseq is like giving your brain a sandbox where every thought, task, and idea can **connect naturally**. Start small:  
- Capture in journals  
- Link generously  
- Slowly build a knowledge garden 🌿

Once you feel comfy, dive into queries, plugins, and custom policies (no, not the federation kind 😉) to turn Logseq into **your personal operating system for knowledge**.

---