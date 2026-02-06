# AI Assistant Feature - Wave 1 Implementation Plan

## Current State
- Research is complete
- Plan is created with 5 waves
- We're starting Wave 1: Fix Critical Issues

## Wave 1 Tasks (in order):\n
1. Fix db import in conversations.js - the file imports `require('../db')` which returns the Database class, but it needs the singleton instance via `Database.getDb()`
2. Check other route files for similar db import issues
3. Create embeddings API routes (generate, batch-generate, search, status) in server/routes/ai.js
4. Test conversation threads work properly