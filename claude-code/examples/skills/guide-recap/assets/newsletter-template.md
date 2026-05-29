> 📚 **AI Spark Wiki** · Claude Code 知识库

# 简报模板

目标：约 500 字。分节展开，有深度。

## 法语模板（FR Template）

```markdown
# {title_fr}

{intro_paragraph_fr}

## Ce qui a change

{highlights_section_fr}

## En detail

{detail_section_fr}

## A retenir

{takeaway_fr}

---

[Guide complet]({landing_url}) | [GitHub]({github_url})
```

## 英语模板（EN Template）

```markdown
# {title_en}

{intro_paragraph_en}

## What changed

{highlights_section_en}

## In detail

{detail_section_en}

## Key takeaway

{takeaway_en}

---

[Full guide]({landing_url}) | [GitHub]({github_url})
```

## 字段规则

### title（最多 80 字符）

以版本号或周次为框架，描述性表达。

```
FR: "Guide v3.20.5 : Reference visuelle enrichie"
EN: "Guide v3.20.5: Enhanced Visual Reference"

FR: "Semaine du 27 janvier : 4 releases, 9 patterns avances"
EN: "Week of January 27: 4 releases, 9 advanced patterns"
```

### intro_paragraph（2-3 句，最多 150 字）

说明发生了什么以及为何重要。不要堆砌噱头。

```
FR: "La version 3.20.5 du Claude Code Ultimate Guide ajoute 4 nouveaux
diagrammes ASCII a la reference visuelle. Le guide contient maintenant
20 diagrammes couvrant TDD, securite et workflows d'apprentissage."

EN: "Version 3.20.5 of the Claude Code Ultimate Guide adds 4 new ASCII
diagrams to the visual reference. The guide now contains 20 diagrams
covering TDD, security, and learning workflows."
```

### highlights_section（项目列表，3-5 条）

从评分最高的条目中提炼转化。每条最多 1-2 句。

```
FR:
- **Cycle TDD Red-Green-Refactor** : Diagramme du flux iteratif test-code-refactor
- **Protocole UVAL** : Visualisation du framework Comprendre-Verifier-Appliquer-Apprendre
- **Defense securite 3 couches** : Prevention, detection, reponse avec guide d'adoption
- **Timeline d'exposition de secrets** : Actions d'urgence par fenetre temporelle (15min/1h/24h)

EN:
- **TDD Red-Green-Refactor Cycle**: Diagram of the iterative test-code-refactor flow
- **UVAL Protocol Flow**: Visualization of the Understand-Verify-Apply-Learn framework
- **Security 3-Layer Defense**: Prevention, detection, response with adoption guide
- **Secret Exposure Timeline**: Emergency actions by time window (15min/1h/24h)
```

### detail_section（1-2 段，最多 200 字）

展开最有价值的亮点内容。提供背景，说明用户能获得什么。如有来源，注明出处。

### takeaway（1-2 句）

单条可执行的洞察或总结。

```
FR: "Si vous apprenez mieux en visuel, les 20 diagrammes du guide couvrent
maintenant les workflows les plus frequents, de TDD a la gestion d'incidents."

EN: "If you're a visual learner, the guide's 20 diagrams now cover the most
common workflows, from TDD to incident management."
```

## 约束条件

- 总字数：400-600 字
- Emoji 配额：2-3 个（仅用于章节标题）
- 禁止使用噱头词汇
- 法语：使用敬语（vouvoiement）
- 英语：美式英语
- 页脚同时包含两个链接（落地页 + GitHub）
- 所有具名来源均需注明
