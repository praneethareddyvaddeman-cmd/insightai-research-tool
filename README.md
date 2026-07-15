# insightai-research-tool
insights project
InsightAI — Consumer Research to Action Platform
Instant consumer insights with specific actionable recommendations for SMBs who can't afford Zappi or Kantar.
Why I Built This
Small brand managers make critical campaign and positioning decisions weekly with no research support. Existing tools deliver scores or reports — none deliver a specific recommendation you can act on today. InsightAI closes that gap.
What It Does
* Takes a natural language research question as input
* Optionally accepts a document for additional context
* Returns a structured insight card: insight, confidence level, recommended action, and sources
* Includes an RLHF-style feedback loop to log outcome data in Supabase
Stack
* n8n — agentic workflow orchestration
* Azure AI Foundry — GPT-4o-mini (synthesis) + GPT-4o (LLM Judge)
* Azure AI Search — RAG grounding and source attribution
* Supabase — feedback and outcome logging
* GitHub Pages — UI
Status: Active build — pushing daily
## Build Log
- Day 1: Question classification (5 categories), SerpAPI web search grounding, GPT-4.1-mini synthesis pipeline working end to end
- Day 2: Feedback webhook working, RLHF loop complete, 
  4 test cases logged in Supabase with helpful ratings
- Day 3: Azure text-embedding-3-small integrated, Content Safety 
  filtering added, full Responsible AI pipeline working end to end
-Day 4:Vercel UI at insightly-beige.vercel.app 
Railway n8n pipeline running 
Azure embeddings + semantic memory 
LLM Judge grounding check 
Azure Content Safety + bias detection 
Supabase RLHF logging 
