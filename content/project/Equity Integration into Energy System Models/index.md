---
date: "2023-09-13T00:00:00Z"
external_link: ""
image:
  caption: From Tennessee Government Website
  focal_point: Smart
#links:
#- icon: twitter
#  icon_pack: fab
#  name: Follow
#  url: https://twitter.com/georgecushen
#slides: example
summary: This article focuses on the modifications and customizations of the Open-Source Energy Modeling System (OSeMOSYS) made for a test case in Austin, Texas.
#tags:
title: Equity Integration into Energy System Models
#url_code: ""
#url_pdf: ""
#url_slides: ""
#url_video: ""
---

This article focuses on the modifications and customizations of the Open-Source Energy Modeling System (OSeMOSYS) made for a test case in Austin, Texas. OSeMOSYS reduces net present costs using a deterministic linear program. It has been used in many situations, such as cross-border electricity trade in South America and capacity planning strategies under climate policy uncertainty. In modifying the standard OSeMOSYS structure, we are developing a General Algebraic Modeling System (GAMS) implementation. The modifications include making POVs non-dispatchable, introducing a formulation of EV charging and vehicle-to-grid (V2G) capabilities, and constraining annual capacity growth rates by technology. This article includes a detailed discussion of the code development process and the results of the simulations using the enhanced code. Given the extensive range of the model output, we concentrate on reporting and interpreting results that are especially pertinent to the economic and environmental effects of SAVs rather than trying to categorize every possible outcome.
