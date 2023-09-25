---
# Leave the homepage title empty to use the site title
title:
date: 2022-10-24
type: landing

sections:
  - block: about.biography
    id: about
    content:
      title: Biography
      # Choose a user profile to display (a folder name within `content/authors/`)
      username: admin
  - block: experience
    content:
      title: Experience
      # Date format for experience
      #   Refer to https://wowchemy.com/docs/customization/#date-format
      date_format: Jan 2006
      # Experiences.
      #   Add/remove as many `experience` items below as you like.
      #   Required fields are `title`, `company`, and `date_start`.
      #   Leave `date_end` empty if it's your current employer.
      #   Begin multi-line descriptions with YAML's `|2-` multi-line prefix.
      items:
        - title: PhD Researcher
          company: Northwestern University
          company_url: ''
          company_logo: 
          location: Evanston, Illinois
          date_start: '2017-09-07'
          date_end: ''
          description: Mechanical characterization of soft materials with optical coherence elastography.
        - title: Hardware Engineering Intern
          company: Apple Inc.
          company_url: ''
          company_logo: 
          location: Cupertino, California
          date_start: '2022-10-03'
          date_end: '2023-07-29'
          description: Support mechanical design of displays with FEA. Focusing on vibration related topics.
    design:
      columns: '2'
  - block: collection
    content:
      title: Recent Publications
      text: |-
        {{% callout note %}}
        Quickly discover relevant content by [filtering publications](./publication/).
        {{% /callout %}}
      filters:
        folders:
          - publication
        exclude_featured: true
    design:
      columns: '2'
      view: citation
---
