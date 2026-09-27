---
kind: playbook
domain: growth
topics: [growth, marketing, aeo]
authority: guidance
---

# AEO — Answer Engine Optimization

Answer Engine Optimization (AEO) is the practice of making content discoverable and citable by AI systems: ChatGPT, Claude, Perplexity, Google AI Overviews, and similar tools that synthesize answers from indexed content rather than returning a list of links.

## Traditional SEO vs. AEO

| Dimension | Traditional SEO | AEO |
|---|---|---|
| **Goal** | Rank on a SERP list | Be cited in an AI-generated answer |
| **Optimization target** | Crawler signals, backlinks, on-page keywords | Clarity, factual density, structured content, authoritativeness |
| **Discovery mechanism** | Users click a blue link | AI summarizes content; user may or may not click through |
| **What gets rewarded** | Relevance + authority signals | Definitive, quotable statements; clear Q&A structure |

The two are complementary, not competing. Content that is well-structured for AEO is also well-structured for search engine understanding. But AEO requires specific content patterns that SEO alone does not.

## Structured Content Techniques for AEO

**1. Question-and-answer format:** AI systems extract explicit answers to questions. Structure content so questions are stated and answered directly.

```markdown
## What is the difference between RBAC and ABAC?

RBAC (Role-Based Access Control) grants permissions based on the user's role.
ABAC (Attribute-Based Access Control) grants permissions based on attributes of both
the user and the resource being accessed.

Use RBAC when permissions are fully described by "what role does this user have?"
Use ABAC when conditional logic appears on top of roles.
```

**2. Clear, concise definitions:** AI systems generate summaries from the first definitional sentence of a section. Make definitional statements precise and self-contained.

**3. FAQ schema markup:** Use `FAQPage` structured data to explicitly signal Q&A content to indexers:

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [{
    "@type": "Question",
    "name": "What is metered billing?",
    "acceptedAnswer": {
      "@type": "Answer",
      "text": "Metered billing charges customers based on their actual usage of a product or service — API calls, data processed, messages sent — rather than a flat recurring fee."
    }
  }]
}
</script>
```

**4. Comparison tables:** AI systems frequently surface comparisons. A well-structured comparison table in a page is likely to be cited when a user asks "X vs. Y?"

## E-E-A-T for AI-Summarized Content

Google's E-E-A-T framework (Experience, Expertise, Authoritativeness, Trustworthiness) was designed for evaluating human-authored content — but LLMs trained on web content absorb these signals too.

- **Experience:** Content that demonstrates first-hand experience (specific examples, real numbers, case studies) is more likely to be cited than generic overview content
- **Expertise:** Author credentials and depth of coverage signal expertise — even to systems that don't explicitly parse author bios
- **Authoritativeness:** Sites cited by other authoritative sources are weighted more heavily in LLM training data
- **Trustworthiness:** Factual accuracy, consistent claims across pages, and agreement with other credible sources

## Practical Guidance

- **Publish comparison and definition content** — these are the query types AI systems most often answer
- **Maintain factual consistency** across all your pages and public social presence — contradictions between pages confuse AI summarization and reduce citation likelihood
- **Use specific, quotable statements** — avoid hedged, vague claims. "Studies suggest that some users may prefer..." is not citable. "Companies using metered billing report 40% lower churn among price-sensitive segments" is citable (if true and sourced)
- **Implement FAQ schema** on pages with explicit Q&A content
- **Link to your content from authoritative sources** — guest posts, community contributions, being cited in third-party content all increase the likelihood your content was included in training data or will be indexed for retrieval

## Related Documents

- [SEO](seo.md)
- [Content marketing](content-marketing.md)
- [Landing page copy](landing-page-copy.md)
