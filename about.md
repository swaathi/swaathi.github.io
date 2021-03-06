---
layout: page
title: About
---

I'm a 27 year old business owner, I've been running [Skcript]({{ site.skcript.main }}) for about {{ site.time | date: '%Y' | minus:2013 }} years now. I work with 17 other [amazing people]({{ site.skcript.people }}) everyday to build awesome things for enterprises.

I fell in love with code when I was 15 years old, and it still hasn't changed. Currently Ruby and Elixir are my addictions. I dabble a bit in infrastructure, a lot in language constructs, and in everything that keeps me curious. Also love to speak about code, tech and startups. There are a few people I admire on the internet, you should [check them out](/people)!

Perpetually trying to make a dent in the universe from the "spicy" side of the equator. Namaste. 🙏

Tweet to me [@{{ site.twitter.username }}]({{ site.author.twitter }}).

<!-- # Values
<dl>
  {% for lesson in site.data.lessons %}
    <dt>{{ lesson.attribute }}</dt>
    <dd>{{ lesson.value }}</dd>
  {% endfor %}
</dl> -->

<center>
  <img src="/public/smile.png" class="img" alt="smile" />
</center>

# Languages and Frameworks

<table>
  <thead>
    <th>Name</th>
    <th>Skill</th>
  </thead>
  <tbody>
    {% for language in site.data.languages %}
      <tr>
        <td>{{ language.name }}</td>
        <td>{{ language.skill }}</td>
      </tr>
    {% endfor %}
  </tbody>
</table>
