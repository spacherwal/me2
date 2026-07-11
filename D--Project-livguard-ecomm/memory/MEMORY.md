# Memory Index

> Memories for this project live inside the repo at `D:\Project\livguard-ecomm\.claude\memory\`.
> Read and write memory files there. This file mirrors the index for auto-load.

- [Ask Before Implementing](D:\Project\livguard-ecomm\.claude\memory\feedback_ask_before_implementing.md) — Present options and wait for user choice before writing code for architectural decisions
- [No any Type](D:\Project\livguard-ecomm\.claude\memory\feedback_no_any_type.md) — Fix TypeScript errors with proper interfaces/constraints, never with `any`
- [No Auto Commits](D:\Project\livguard-ecomm\.claude\memory\feedback_no_auto_commit.md) — Never commit automatically; user controls all git commits
- [No Commits or Pushes](D:\Project\livguard-ecomm\.claude\memory\feedback_no_commit_push.md) — Make changes locally only; skip all git add/commit/push steps unless user explicitly asks
- [No Build Check via CLI](D:\Project\livguard-ecomm\.claude\memory\feedback_no_build_check.md) — User verifies compilation in IDE; don't run npm run build via terminal
- [Memory Scope](D:\Project\livguard-ecomm\.claude\memory\feedback_memory_scope.md) — All memories live inside the repo at .claude/memory/, not at C: user path
- [CartContext Architecture](D:\Project\livguard-ecomm\.claude\memory\project_cart_context.md) — CartContext owns all cart state; single API call shared by navbar and cart page via useCart()
- [Razorpay Standard SDK](D:\Project\livguard-ecomm\.claude\memory\project_razorpay_standard_sdk.md) — Full context: architecture, file map, decisions, navigationArea mapping, UPI Collect deprecated Feb 2026
- [Bitbucket Pipeline .env.local Gap](D:\Project\livguard-ecomm\.claude\memory\project_bitbucket_env_lesson.md) — Every new env var must be added to both Bitbucket repo variables AND the heredoc in bitbucket-pipelines.yml
- [CICD Debugging Lesson Apr 21 2026](D:\Project\livguard-ecomm\.claude\memory\project_cicd_lesson_apr21.md) — CodeDeploy bootstrap loop fix + npm ci lock file mismatch on EC2 Ubuntu
- [PDP Pincode Issue](D:\Project\livguard-ecomm\.claude\memory\project_pdp_pincode_issue.md) — /api/web/offers.json requires x-pincode header; missing cookie → "Product not found" for all PDPs; backend fix pending (2026-05-12)
- [Server Auth Token Plan](D:\Project\livguard-ecomm\.claude\memory\project_server_auth_plan.md) — Pending: fix auth token unavailable in server components; 3 options discussed, not yet implemented
- [Products Page Implementation](D:\Project\livguard-ecomm\.claude\memory\project_products_page.md) — /products page with API integration, Shop Now button wired up, TopBar/Navbar/Footer included
- [me2 Repo](D:\Project\livguard-ecomm\.claude\memory\reference_me2_repo.md) — GitHub repo where session learnings are pushed: github.com/spacherwal/me2
- [Design Before Implement](D:\Project\livguard-ecomm\.claude\memory\feedback_design_before_implement.md) — For UI redesigns, present mockup/plan first and wait for approval before writing code
