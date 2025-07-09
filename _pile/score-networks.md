---
title: Score networks
description: The vector field of a score network
# badge_text: "danger"
# badge_color: "danger"
date: 2025-07-09
math: true
---

<!-- Epdata(x)
tr(∇xsθ(x)) + 1/2 ksθ(x)k^2 -->


This is the objective function for training a score-based model via score matching.

$$
\Large
\mathbb{E}_{p_{\text{data}}(\mathbf{x})}\left[\mathrm{tr}(\nabla_{\mathbf{x}}\mathbf{s}_{\theta}(\mathbf{x})) + \frac{1}{2}\|\mathbf{s}_{\theta}(\mathbf{x})\|_{2}^{2}\right]
$$

Sampling via Langevin dynamics is given by:

$$
\Large
\tilde{\mathbf{x}}_t = \tilde{\mathbf{x}}_{t-1} + \frac{\epsilon}{2} \nabla_{\mathbf{x}} \log p(\tilde{\mathbf{x}}_{t-1}) + \sqrt{\epsilon} \mathbf{z}_t
$$


![Score networks](https://private-user-images.githubusercontent.com/48069158/455284817-5affd982-6646-48cb-bd39-88714be8a80e.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTIwMzc1NTEsIm5iZiI6MTc1MjAzNzI1MSwicGF0aCI6Ii80ODA2OTE1OC80NTUyODQ4MTctNWFmZmQ5ODItNjY0Ni00OGNiLWJkMzktODg3MTRiZThhODBlLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTA3MDklMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwNzA5VDA1MDA1MVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPThjNTNjMjVlNWIyNmYwMzFlNTNhMzEwZWNkNjc4MWM4MWI0N2M4YzM5ZmFkYjZiMDRhNmZkZWQzNTYxZGJjMjMmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.5SI1qYtZWQMQGDtV_C8C8YY-F1zbAk9LHxi8t3o-MG8)

![Langevin sampling](https://private-user-images.githubusercontent.com/48069158/455284727-ea38069a-4457-4d7e-8b23-97f1e08f143f.gif?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTIwMzc1NTEsIm5iZiI6MTc1MjAzNzI1MSwicGF0aCI6Ii80ODA2OTE1OC80NTUyODQ3MjctZWEzODA2OWEtNDQ1Ny00ZDdlLThiMjMtOTdmMWUwOGYxNDNmLmdpZj9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTA3MDklMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwNzA5VDA1MDA1MVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTE2NzRlZWM3ZmI2OTQ4ZWVkZjUwMzhjODJiOTllOTRhMDc1NzJmYmE3Yzc2NWE4ODNkNmRhMTdjMjY1MmExNDEmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.yn7VXmkxiFTmU2_WwC1TbjfTvXQyLIiaFnD7uFoaq7o)