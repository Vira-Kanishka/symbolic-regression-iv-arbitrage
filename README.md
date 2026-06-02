# Static Arbitrage of Symbolic-Regression IV Parametrizations

A test of whether the symbolic-regression parametrizations of the
implied-volatility smile from Keller-Ressel & Nikulski (2026), in particular
$f_4$, are free of slice-level static arbitrage, which the paper
explicitly leaves open.

## Question

Keller-Ressel & Nikulski (2026) use symbolic regression to discover analytic
parametrizations of the implied-volatility smile that attain lower complexity
and lower fitting loss than SVI on empirical data. The form they single out is

$$f_4(x) = p_4\big(\tanh(p_1 x) + p_2\big)(p_3 + x) + p_5,$$

where $x = \ln(K/F)$ denotes log-moneyness ($K$ strike, $F$ forward) and
$f_4(x)$ models the total implied variance $w(x) = \sigma_{\mathrm{BS}}(x)^2\,\tau$
at maturity $\tau$. A static no-arbitrage analysis of $f_4$ is left open in the
paper. Keller-Ressel & Nikulski (2026) also present two extended forms, C6 and C8 (equations
4.2 and 4.3),

$$\widehat{w}^{C_6}(x) = p_1 + p_2 x + p_3\sqrt{(x - p_4)^2 + p_5^2} + p_6\sqrt{(x - p_7)^2 + p_8^2}$$

$$\widehat{w}^{C_8}(x) = p_1 + p_2 x + p_3\sqrt{(x - p_4)^2 + p_5^2} + \sqrt{x^2 + p_6} + p_7\, e^{-(x - p_8)^2}$$

which extend SVI with a second hyperbolic term (C6) or a Gaussian bump (C8)
to fit smile shapes the five-parameter SVI form cannot reproduce. This
notebook supplies the static-arbitrage analysis, applying standard
slice-level no-arbitrage conditions to SVI, $f_4$, C6, and C8.



## What the notebook does

1. **Implementation:** Four parametrizations (SVI, $f_4$, C6, C8) and the
   slice-level no-arbitrage conditions: positivity, Durrleman's $g \ge 0$,
   Lee's tail bound $\limsup_{x \to \pm\infty} w(x)/|x| \le 2$, and the
   quantitative arbitrage indicator $\alpha_K$.

2. **Validation:** The diagnostics are first verified on two slices with known
   arbitrage status: an arbitrage-free SVI slice, which should pass, and the
   Vogt smile of Gatheral and Jacquier (2014), the canonical
   butterfly-arbitrageable example, which should be flagged.

3. **Applications:** The diagnostics are then applied to (i) a representative
   SVI-generated slice with quote noise, (ii) a bimodal-mixture smile from a
   two-component lognormal mixture with martingale-consistent component
   forwards, and (iii) a live Deribit BTC slice retrieved through the public
   market-data endpoint.

## Results

**Validation.** The arbitrage-free SVI slice satisfies all conditions
($\min g = 0.25$, $\alpha_K = 0$). The Vogt smile is correctly flagged
($\min g = -0.033$, $\alpha_K = 0.013$), with the violation identified in the
same region of log-moneyness by both $g(x)$ and the independent
Breeden–Litzenberger density.

**Representative slice.** Both SVI and $f_4$ satisfy all conditions. SVI
attains the lowest fitting error by construction (0.33 against 0.35
implied-volatility points), as expected when the target is generated from SVI.
$f_4$'s arbitrage status, left open in the paper, is here determined: it
remains free of butterfly arbitrage on this slice.

**Bimodal-mixture slice.** SVI's fit attains 0.86 implied-volatility points
with a substantial butterfly violation ($\min g = -5.7$). $f_4$'s fit attains
0.60 with a mild butterfly violation ($\min g = -0.03$). C6 attains 0.04 and
satisfies both Durrleman's condition and the Lee tail bound; C8 attains 0.30
and satisfies Durrleman's condition (its fitted tails exceed the Lee bound on
extrapolation).

**Live Deribit BTC slice** ($T \approx 0.07$, 118 quotes). $f_4$ and C8 reduce
the fitting error to roughly half of SVI's (8.7 and 6.4 implied-volatility
points against SVI's 17.0), with all four parametrizations butterfly-arbitrage-free
on the quoted range — the paper's accuracy claim for the discovered forms
reproduced on market data.

## Running

```
pip install -r requirements.txt
jupyter notebook svi_arbitrage_stresstest.ipynb
```

The cells run top to bottom. The Deribit cell (Section 7) requires network
access; the rest run offline.

## Limitations

The diagnostic is slice-level (butterfly and Lee's tail bound), so the
conclusion is per-maturity rather than surface-wide. Calendar arbitrage
requires fitting each form across maturities and verifying that total variance
is non-decreasing in $\tau$ at fixed $x$; this is the natural extension.
Durrleman's $g \ge 0$ with positivity is necessary and sufficient for SVI
(Martini and Mingone, 2022) but only necessary and close to sufficient for the
discovered forms (Roper, 2009), so a passing diagnostic is strong evidence of
admissibility for these forms, not a proof of it.

## References

Gatheral, J., & Jacquier, A. (2014). Arbitrage-free SVI volatility surfaces.
*Quantitative Finance*, *14*(1), 59–71.
https://doi.org/10.1080/14697688.2013.819986

Glasserman, P., & Pirjol, D. (2023). W-shaped implied volatility curves and the
Gaussian mixture model. *Quantitative Finance*, *23*(4), 557–577.
https://doi.org/10.1080/14697688.2023.2165448

Keller-Ressel, M., & Nikulski, H. (2026). *Discovering parametrizations of
implied volatility with symbolic regression* (arXiv:2603.21892). arXiv.
https://doi.org/10.48550/arXiv.2603.21892

Lee, R. W. (2004). The moment formula for implied volatility at extreme strikes.
*Mathematical Finance*, *14*(3), 469–480.
https://doi.org/10.1111/j.0960-1627.2004.00200.x

Martini, C., & Mingone, A. (2022). No arbitrage SVI. *SIAM Journal on Financial
Mathematics*, *13*(1), 227–261. https://doi.org/10.1137/20M1351060

Roper, M. P. V. (2009). *Implied volatility: General properties and asymptotics*
[Doctoral dissertation, University of New South Wales]. UNSWorks.
https://doi.org/10.26190/unsworks/21796
