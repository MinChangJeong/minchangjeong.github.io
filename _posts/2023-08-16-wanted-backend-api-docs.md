---
layout: post
title:  "원티드 백엔드 선발 과제 - API Documentation"
date:   2023-08-16
excerpt: "wanted-pre-onboarding-backend API Documentation"
project: true
tags: [springboot, docker, junit]
comments: true
---


<html lang="en">
<head>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.7.0/css/font-awesome.min.css">
</head>
<body class="book toc2 toc-left">
<div id="header">
<h1>Wanted API Documentation</h1>
<div id="toc" class="toc2">
<div id="toctitle">Table of Contents</div>
<ul class="sectlevel1">
<li><a href="#_member_api">Member API</a>
<ul class="sectlevel2">
<li><a href="#_로그인">로그인</a>
<ul class="sectlevel3">
<li><a href="#_로그인_http_request">HTTP request</a></li>
<li><a href="#_로그인_http_response">HTTP response</a></li>
</ul>
</li>
<li><a href="#_회원_등록">회원 등록</a>
<ul class="sectlevel3">
<li><a href="#_회원_등록_http_request">HTTP request</a></li>
<li><a href="#_회원_등록_http_response">HTTP response</a></li>
</ul>
</li>
</ul>
</li>
<li><a href="#_board_api">Board API</a>
<ul class="sectlevel2">
<li><a href="#_게시글_등록">게시글 등록</a>
<ul class="sectlevel3">
<li><a href="#_게시글_등록_http_request">HTTP request</a></li>
<li><a href="#_게시글_등록_http_response">HTTP response</a></li>
</ul>
</li>
<li><a href="#_게시글_목록_조회">게시글 목록 조회</a>
<ul class="sectlevel3">
<li><a href="#_게시글_목록_조회_http_request">HTTP request</a></li>
<li><a href="#_게시글_목록_조회_http_response">HTTP response</a></li>
</ul>
</li>
<li><a href="#_게시글_상세_조회">게시글 상세 조회</a>
<ul class="sectlevel3">
<li><a href="#_게시글_상세_조회_http_request">HTTP request</a></li>
<li><a href="#_게시글_상세_조회_http_response">HTTP response</a></li>
</ul>
</li>
<li><a href="#_게시글_수정">게시글 수정</a>
<ul class="sectlevel3">
<li><a href="#_게시글_수정_http_request">HTTP request</a></li>
<li><a href="#_게시글_수정_http_response">HTTP response</a></li>
</ul>
</li>
<li><a href="#_게시글_삭제">게시글 삭제</a>
<ul class="sectlevel3">
<li><a href="#_게시글_삭제_http_request">HTTP request</a></li>
<li><a href="#_게시글_삭제_http_response">HTTP response</a></li>
</ul>
</li>
</ul>
</li>
</ul>
</div>
</div>
<div id="content">
<div class="sect1">
<h2 id="_member_api"><a class="link" href="#_member_api">Member API</a></h2>
<div class="sectionbody">
<div class="sect2">
<h3 id="_로그인"><a class="link" href="#_로그인">로그인</a></h3>
<div class="sect3">
<h4 id="_로그인_http_request"><a class="link" href="#_로그인_http_request">HTTP request</a></h4>
<div class="listingblock">
<div class="content">
<pre class="highlight nowrap"><code class="language-http" data-lang="http">POST /member/login HTTP/1.1
Content-Type: application/json
Content-Length: 68
Host: localhost:8080

{
  "email" : "wjdalsckd777@naver.com",
  "password" : "test12345"
}</code></pre>
</div>
</div>
</div>
<div class="sect3">
<h4 id="_로그인_http_response"><a class="link" href="#_로그인_http_response">HTTP response</a></h4>
<div class="listingblock">
<div class="content">
<pre class="highlight nowrap"><code class="language-http" data-lang="http">HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 35

{
  "accessToken" : "accessToken"
}</code></pre>
</div>
</div>
</div>
</div>
<div class="sect2">
<h3 id="_회원_등록"><a class="link" href="#_회원_등록">회원 등록</a></h3>
<div class="sect3">
<h4 id="_회원_등록_http_request"><a class="link" href="#_회원_등록_http_request">HTTP request</a></h4>
<div class="listingblock">
<div class="content">
<pre class="highlight nowrap"><code class="language-http" data-lang="http">POST /member HTTP/1.1
Content-Type: application/json
Content-Length: 68
Host: localhost:8080

{
  "email" : "wjdalsckd777@naver.com",
  "password" : "test12345"
}</code></pre>
</div>
</div>
</div>
<div class="sect3">
<h4 id="_회원_등록_http_response"><a class="link" href="#_회원_등록_http_response">HTTP response</a></h4>
<div class="listingblock">
<div class="content">
<pre class="highlight nowrap"><code class="language-http" data-lang="http">HTTP/1.1 200 OK</code></pre>
</div>
</div>
</div>
</div>
</div>
</div>
<div class="sect1">
<h2 id="_board_api"><a class="link" href="#_board_api">Board API</a></h2>
<div class="sectionbody">
<div class="sect2">
<h3 id="_게시글_등록"><a class="link" href="#_게시글_등록">게시글 등록</a></h3>
<div class="sect3">
<h4 id="_게시글_등록_http_request"><a class="link" href="#_게시글_등록_http_request">HTTP request</a></h4>
<div class="listingblock">
<div class="content">
<pre class="highlight nowrap"><code class="language-http" data-lang="http">POST /board HTTP/1.1
Content-Type: application/json
Authorization: Bearer accessToken
Content-Length: 72
Host: localhost:8080

{
  "title" : "게시글 제목 1",
  "content" : "게시글 내용 1"
}</code></pre>
</div>
</div>
</div>
<div class="sect3">
<h4 id="_게시글_등록_http_response"><a class="link" href="#_게시글_등록_http_response">HTTP response</a></h4>
<div class="listingblock">
<div class="content">
<pre class="highlight nowrap"><code class="language-http" data-lang="http">HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 75

{
  "message" : "게시글 등록을 성공적으로 완료했습니다."
}</code></pre>
</div>
</div>
</div>
</div>
<div class="sect2">
<h3 id="_게시글_목록_조회"><a class="link" href="#_게시글_목록_조회">게시글 목록 조회</a></h3>
<div class="sect3">
<h4 id="_게시글_목록_조회_http_request"><a class="link" href="#_게시글_목록_조회_http_request">HTTP request</a></h4>
<div class="listingblock">
<div class="content">
<pre class="highlight nowrap"><code class="language-http" data-lang="http">GET /board?page=0&amp;size=10 HTTP/1.1
Host: localhost:8080</code></pre>
</div>
</div>
</div>
<div class="sect3">
<h4 id="_게시글_목록_조회_http_response"><a class="link" href="#_게시글_목록_조회_http_response">HTTP response</a></h4>
<div class="listingblock">
<div class="content">
<pre class="highlight nowrap"><code class="language-http" data-lang="http">HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 671

{
  "content" : [ {
    "boardId" : 1,
    "title" : "게시글 제목 1",
    "content" : "게시글 내용 1",
    "createdAt" : "2023-08-16T00:18:10.41149",
    "updatedAt" : "2023-08-16T00:18:10.411502"
  }, {
    "boardId" : 1,
    "title" : "게시글 제목 1",
    "content" : "게시글 내용 1",
    "createdAt" : "2023-08-16T00:18:10.411506",
    "updatedAt" : "2023-08-16T00:18:10.411508"
  } ],
  "pageable" : "INSTANCE",
  "last" : true,
  "totalPages" : 1,
  "totalElements" : 2,
  "first" : true,
  "size" : 2,
  "number" : 0,
  "sort" : {
    "empty" : true,
    "unsorted" : true,
    "sorted" : false
  },
  "numberOfElements" : 2,
  "empty" : false
}</code></pre>
</div>
</div>
</div>
</div>
<div class="sect2">
<h3 id="_게시글_상세_조회"><a class="link" href="#_게시글_상세_조회">게시글 상세 조회</a></h3>
<div class="sect3">
<h4 id="_게시글_상세_조회_http_request"><a class="link" href="#_게시글_상세_조회_http_request">HTTP request</a></h4>
<div class="listingblock">
<div class="content">
<pre class="highlight nowrap"><code class="language-http" data-lang="http">GET /board/1 HTTP/1.1
Host: localhost:8080</code></pre>
</div>
</div>
</div>
<div class="sect3">
<h4 id="_게시글_상세_조회_http_response"><a class="link" href="#_게시글_상세_조회_http_response">HTTP response</a></h4>
<div class="listingblock">
<div class="content">
<pre class="highlight nowrap"><code class="language-http" data-lang="http">HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 181

{
  "boardId" : 1,
  "title" : "게시글 제목 1",
  "content" : "게시글 내용 1",
  "createdAt" : "2023-08-16T00:18:10.379776",
  "updatedAt" : "2023-08-16T00:18:10.379794"
}</code></pre>
</div>
</div>
</div>
</div>
<div class="sect2">
<h3 id="_게시글_수정"><a class="link" href="#_게시글_수정">게시글 수정</a></h3>
<div class="sect3">
<h4 id="_게시글_수정_http_request"><a class="link" href="#_게시글_수정_http_request">HTTP request</a></h4>
<div class="listingblock">
<div class="content">
<pre class="highlight nowrap"><code class="language-http" data-lang="http">PUT /board HTTP/1.1
Content-Type: application/json
Authorization: Bearer accessToken
Content-Length: 89
Host: localhost:8080

{
  "boardId" : 1,
  "title" : "게시글 제목 1",
  "content" : "게시글 내용 1"
}</code></pre>
</div>
</div>
</div>
<div class="sect3">
<h4 id="_게시글_수정_http_response"><a class="link" href="#_게시글_수정_http_response">HTTP response</a></h4>
<div class="listingblock">
<div class="content">
<pre class="highlight nowrap"><code class="language-http" data-lang="http">HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 75

{
  "message" : "게시글 수정을 성공적으로 완료했습니다."
}</code></pre>
</div>
</div>
</div>
</div>
<div class="sect2">
<h3 id="_게시글_삭제"><a class="link" href="#_게시글_삭제">게시글 삭제</a></h3>
<div class="sect3">
<h4 id="_게시글_삭제_http_request"><a class="link" href="#_게시글_삭제_http_request">HTTP request</a></h4>
<div class="listingblock">
<div class="content">
<pre class="highlight nowrap"><code class="language-http" data-lang="http">DELETE /board/1 HTTP/1.1
Authorization: eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMDAxIiwicm9sZXMiOltdLCJpYXQiOjE2MTA4MTA2NDEsImV4cCI6MTYxMDg5NzA0MX0.5rHP_aEyJk0wpozI1Azy1LAJZzIx0ZLG5ZDQrKrP8EM
Host: localhost:8080</code></pre>
</div>
</div>
</div>
<div class="sect3">
<h4 id="_게시글_삭제_http_response"><a class="link" href="#_게시글_삭제_http_response">HTTP response</a></h4>
<div class="listingblock">
<div class="content">
<pre class="highlight nowrap"><code class="language-http" data-lang="http">HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 75

{
  "message" : "게시글 삭제를 성공적으로 완료했습니다."
}</code></pre>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<div id="footer">
<div id="footer-text">
Last updated 2023-08-14 10:53:03 +0900
</div>
</div>
</body>
</html>