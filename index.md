---
title: オンライン ホステッド インストラクション
permalink: index.html
layout: home
---

# コンテンツ ディレクトリ

このリポジトリには、Microsoft のコース [AI-102 Microsoft Azure AI ソリューションの設計と実装](https://docs.microsoft.com/learn/certifications/courses/ai-102t00) 実践的な演習と、同等の [Microsoft Learn のマイペースで進められるモジュール](https://aka.ms/AzureLearn_AIEngineer-jpn) が含まれています。演習は学習教材に沿って設計されており、教材で説明されているテクノロジを使って学習できます。

この演習を完了するには、Microsoft Azure サブスクリプションが必要です。講師からサブスクリプションが提供されていない場合は、[https://azure.microsoft.com](https://azure.microsoft.com) で無料の試用版にサインアップできます。

## ラボ

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions'" %}
| モジュール | ラボ |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

