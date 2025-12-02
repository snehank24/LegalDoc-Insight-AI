# LegalDoc-Insight-AI

# LegalDoc Insight – AI-Powered Document Review Assistant

**Building AI course project**

## Summary
LegalDoc Insight is an AI-powered tool that automates summarization, categorization, and risk flagging of legal documents to speed up litigation review processes and improve accuracy. It helps legal teams save time and reduce errors in document review.

---

## Background time-consuming and error-prone. Review managers and teams spend hours tagging relevance, privilege, and issue categories manually. Missing key clauses or misclassifying documents can lead to compliance risks and costly errors.## Background

Problems this idea solves:
* Manual document review takes too much time.
* High risk of human error in tagging relevance and privilege.
* Increased costs for law firms and corporations.

My motivation: I have managed document review projects for years and seen inefficiencies firsthand. This topic is important because AI can drastically reduce review time, improve consistency, and allow legal professionals to focus on strategic tasks.

---

## How is it used?
The solution is used during litigation and compliance audits. Review managers and attorneys upload documents to the system, which then:
* Summarizes documents.
* Predicts relevance and privilege status.
* Flags risky clauses and identifies key entities.

Users:
* Review Managers
* Review Attorneys
* Compliance Teams
* Corporate Legal Departments

Example image placeholder:
!Legal Documents

---

## Data sources and AI methods
Data sources:
* Historical reviewed documents with coding tags (relevance, privilege, issue codes).
* Legal text from contracts, emails, memos.
* Metadata (dates, authors, recipients).

AI methods:
* Natural Language Processing (NLP) for summarization and clause extraction.
* Classification models for relevance and privilege.
* Named Entity Recognition (NER) for identifying key individuals.
* Topic Modeling for grouping documents by issues.

| AI Technique      | Purpose |
| ----------------- | ------- |
| NLP Summarization | Reduce document length |
| Classification    | Predict relevance/privilege |
| NER               | Identify people/entities |
| Topic Modeling    | Group documents by issues |

---

## Challenges
* Cannot fully replace human judgment for nuanced privilege calls.
* Requires high-quality training data to avoid bias.
* Sensitive legal data raises privacy and security concerns.

---

## What next?
* Expand to multilingual document review.
* Integrate with eDiscovery platforms.
* Add predictive analytics for litigation risk assessment.
* Skills needed: advanced NLP, data engineering, cybersecurity expertise.

---

## Acknowledgments
* Inspiration from Relativity and Everlaw.
* Open-source NLP libraries: spaCy, Hugging Face Transformers.
* Sleeping Cat on Her Back by Umberto Salvagnin / CC BY 2.0

* 
# Minimal demo snippet: summarize, extract entities, flag risks
# Requires: transformers, spacy, and the spaCy model `en_core_web_sm`
# Fallbacks included if models aren't available.

import re

def fallback_summarize(text, max_sentences=3):
    sentences = re.split(r"(?<=[.!?])\s+", text.strip())
    keywords = ["agreement","clause","indemnity","privilege","confidential",
                "compliance","risk","non-compete","cross-border","data"]
    scored = sorted(
        [(sum(s.lower().count(k) for k in keywords), i, s) for i, s in enumerate(sentences)],
        key=lambda x: (-x[0], x[1])
    )
    return " ".join(s for _, _, s in sorted(scored[:max_sentences], key=lambda x: x[1]))

def summarize(text):
    try:
        from transformers import pipeline
        pipe = pipeline("summarization", model="facebook/bart-large-cnn")
        return pipe(text, max_length=160, min_length=60, do_sample=False)[0]["summary_text"]
    except Exception:
        return fallback_summarize(text)

def extract_entities(text):
    try:
        import spacy
        nlp = spacy.load("en_core_web_sm")
        doc = nlp(text)
        allowed = {"PERSON","ORG","GPE","DATE","LAW","NORP","FAC","LOC","PRODUCT","EVENT"}
        return [(ent.text, ent.label_) for ent in doc.ents if ent.label_ in allowed]
    except Exception:
        # Simple fallback: capitalized phrases
        caps = set(re.findall(r"\b([A-Z][a-z]+(?:\s+[A-Z][a-z]+)*)\b", text))
        return [(c, "CAP") for c in caps]

def flag_risks(text):
    patterns = {
        "Non-compete": [r"non\s*-?compete"],
        "Indemnity cap": [r"indemnity", r"cap"],
        "Cross-border transfers": [r"cross\s*-?border", r"transfer"],
        "Confidential Information": [r"confidential\s+information"],
        "Privilege": [r"privilege|privileged"],
    }
    lower = text.lower()
    return {label: any(re.search(p, lower) for p in pats) for label, pats in patterns.items()}

if __name__ == "__main__":
    text = """Subject: Contract Negotiation – Confidential

    The draft Services Agreement includes a non-compete clause, a DPA, and an indemnity cap at 1x fees.
    Compliance flagged potential exposure around cross-border transfers and undefined 'Confidential Information'.
    Confirm whether communications with Legal Ops are privileged.
    """

    summary = summarize(text)
    entities = extract_entities(text)
    risks = flag_risks(text)

    print("\n=== Summary ===\n", summary)
    print("\n=== Entities ===")
    for t, lbl in entities: print(f"- {t} [{lbl}]")
    print("\n=== Risk Flags ===")
    for k, v in risks.items(): print(f"- {k}: {'YES' if v else 'NO'}")

