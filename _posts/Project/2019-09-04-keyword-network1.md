---
layout: post
title:  "Summarizing AAG conference abstracts and keywords using keywords network analysis (1)"
date:   2019-09-04 11:40:00
author: Youngho Lee
categories: Project
cover:  "/assets/project/wordcloud.PNG"
---

본 포스팅은 과학기술정보통신부와 정보통신기획평가원이 주관하는 "2019년 글로벌 핵심인재 양성지원 사업"의 프로젝트 과정에서 사용한 코드들을 정리하기 위해 작성하였다. 프로젝트 주제는 "컨퍼런스 최신 연구 동향 분석 : 언어네트워크 분석과 빅데이터 분석을 적용한 Generic Platform 개발"로, Association for American Geographers(AAG, http://aag.org) 컨퍼런스에 1998~2019년간 투고된 논문들의 초록과 키워드들을 사례로 지리학의 연구경향을 파악하고자 하는 연구이다. AAG 컨퍼런스는 매년 5일간 약 7천여명의 연구자들이 등록하는 대규모 학술공동체이며, GIS·인문지리·환경지리·지형학·교통지리·원격탐사 등 지리학의 다양한 분야의 연구들이 매년 소개되고 있다.
이번 글은 프로젝트의 첫 번째 세부프로젝트인 "키워드 네트워크 분석을 활용한 AAG 컨퍼런스 요약"에 관한 글로 첫 단계인 데이터 전처리 코드에 대해 정리할 것이다.