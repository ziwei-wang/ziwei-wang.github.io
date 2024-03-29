---
# Leave the homepage title empty to use the site title
title:
date: 2023-09-26
type: landing

sections:
  - block: about.biography
    id: about
    content:
      title: Biography
      # Choose a user profile to display (a folder name within `content/authors/`)
      username: admin
  - block: experience
    id: exp
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
        - title: Hardware Engineering Intern
          company: Apple Inc.
          company_url: 'https://www.apple.com'
          company_logo: Apple_logo_black
          location: Cupertino, California
          date_start: '2022-10-03'
          date_end: '2023-07-29'
          description: Worked in Display FEA team to support mechanical designs of displays on MacBooks, iPads and other Apple products.
        - title: PhD Researcher
          company: Northwestern University
          company_url: ''
          company_logo: Northwestern_University_seal
          location: Evanston, Illinois
          date_start: '2017-09-07'
          date_end: ''
          description: Focusing on developing mechanical characterization methods based on optical coherence elastography (OCE) for soft materials.
        - title: Graduate Student Researcher
          company: Beijing Institute of Technology
          company_url: ''
          company_logo: BIT
          location: Beijing, China
          date_start: '2014-08-03'
          date_end: '2017-06-29'
          description: Focusing on digital elastic metamaterial design for mechanical vibration control.
    design:
      columns: '2'
  - block: collection
    id: featured
    content:
      title: Featured Publications
      filters:
        folders:
          - publication
        featured_only: true
    design:
      columns: '2'
      view: card
  - block: collection
    id: pub
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