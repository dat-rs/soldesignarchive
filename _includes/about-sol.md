{% assign published_artifacts = site.artifacts | where: 'published', 'true' %}
{% assign published_authors = site.authors | where: 'published', 'true' %}

<p class="about-sol">
Sol Design Archive is an online archive of design with a focus on 20th century Portuguese graphic design and illustration but a voracious appetite for the full breadth of visual expression. The online archive currently features <strong>{{ published_artifacts.size }}</strong> artifacts by <strong>{{ published_authors.size }}</strong> authors. Collected, maintained, and designed by Ricardo Santos out of love for these shapes and colors.
</p>
