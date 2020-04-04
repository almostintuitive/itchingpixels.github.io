---
layout: post
title: Me
---

If we were to meet in person, I'd be more like: [what do you do?](mailto:mark.szulyovszky@gmail.com)

But fine, I'll tell you about myself.

Or... I thought a timeline of what I've done could be a fun way to represent this better than words:

{{< mermaid >}}

stateDiagram
  [*] --> HackingThroughoutMyTeenageYears
  HackingThroughoutMyTeenageYears --> SoundEngineering
  state fork_state
    SoundEngineering --> University
    SoundEngineering --> ProfessionalOnlinePokerPlayer

  state join_state
    University --> Designer
    University --> BehaviouralSciences
    BehaviouralSciences --> Designer
    ProfessionalOnlinePokerPlayer --> Designer
  FormingMyVenture --> SoftwareEngineering
  Designer --> SoftwareEngineering
  Designer --> MovingToLondon
  MovingToLondon --> SoftwareEngineering
  Designer --> FormingMyVenture
  FormingMyVenture --> MovingToLondon
  HackingThroughoutMyTeenageYears --> SoftwareEngineering
  SoftwareEngineering --> CTOofDrops
  CTOofDrops --> [*]
{{< /mermaid >}}