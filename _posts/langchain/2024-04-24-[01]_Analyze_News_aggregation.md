---
permalink: /langchain/Analyze_News_aggregation/
title: "모아 놓은 News 분석 시키기"
excerpt: "News를 스케쥴에 따라서 분석하도록 프로그램 조정"
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - langchain
tags:
  - 자동화Bot
  - AI Agent
---

## 배경과 목표

1시간에 한번씩 News를 모아 놓았으니 LLM으로 요약을 시켜서 저장해보자.

## 구현

- 기존에 Agent를 위해서 Tool을 만들어 놓은 것을 그대로 이용해서 Main함수에서 그대로 사용하였다.
- Keyword 확인을 위하여 DB를 Access한다.

### 
