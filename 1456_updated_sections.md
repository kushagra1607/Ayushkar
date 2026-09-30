# ctp_scicode_material_science_1456 — updated sections (v5)

Replace the cells below in the 1456 notebook. Every other cell (Metadata, Title,
Main Problem prompt/background/tests) stays exactly as it is.
`ctp_scicode_material_science_1456_5.ipynb` in this folder already has all of them applied.

| Unit | Cell | Change |
|---|---|---|
| Subproblem 1 | Prompt | "Uncertainties" paragraph now gives the propagation formula explicitly; tolerances + time limit stated |
| Subproblem 1 | Background | "Counting statistics" paragraph: why the end-point counts dominate at small L |
| Subproblem 1 | Testing Template | `sigma_A` tolerance 1e-6 -> 1e-5 (expected values unchanged) |
| Subproblem 1 | Solution | **unchanged** |
| Subproblem 2 | Prompt | new step 6 (integral breadth + FWHM of predicted line profile), ValueError conditions, tolerances |
| Subproblem 2 | Background | new "Line-profile widths" paragraph + Langford/Louër/Scardi reference |
| Subproblem 2 | Testing Template | 8 outputs checked; new test 7 (invalid inputs) and test 8 (broad distribution) |
| Subproblem 2 | Solution | new |
| Main Problem | Solution | contains the new subproblem_2; `main_problem` only takes `[:5]` of it (outputs identical, all 6 tests pass) |

---

## Subproblem 1 — Prompt

````markdown
Compute two quantities for one X-ray reflection recorded in laboratory \\(2\theta\\) step scans: the Fourier cosine coefficients \\(A(L)\\) of its physical line profile, corrected for instrumental broadening by the Stokes method and for the Kα2 component of the radiation, and the standard uncertainties \\(\sigma_A(L)\\) of these coefficients due to counting statistics. The function returns both, as the tuple \\((A, \sigma_A)\\).

Two step scans of the same reflection are supplied. The specimen scan, with profile \\(h\\), was recorded with Kα1/Kα2 radiation. The instrumental profile \\(g\\) belongs to a strain-free, coarse-grained standard at the same reflection position and contains the Kα1 component only. Let \\(f\\) denote the physical line profile of the specimen for Kα1 radiation. Each scan is a list of counts at increasing \\(2\theta\\) positions in degrees, and the counts are an intensity per unit \\(2\theta\\). The two scans may cover different angular ranges with different, not necessarily uniform, step sizes, and the counts include a background and, in general, counting noise.

Treat each of the two scans independently. First subtract the straight line in \\(2\theta\\) that passes through the first and the last recorded points of the scan. Then express every recorded point in the reciprocal-space coordinate \\(s = \dfrac{2}{\lambda_1}\left(\sin\theta - \sin\theta_0\right)\\) in nm\\(^{-1}\\), where \\(\theta\\) is half the scattering angle, \\(\lambda_1\\) is the Kα1 wavelength and \\(2\theta_0\\) is the reference angle (the Kα1 peak position of the reflection), which is the same for both scans. The Fourier analysis is carried out on the background-corrected profile expressed as an intensity per unit \\(s\\), \\(I_s(s)\\), which is taken to vary linearly in \\(s\\) between neighbouring recorded points and to vanish outside the scan. For a column length \\(L\\) in nm, the normalised cosine and sine coefficients of a scan are

\\[C(L) = \frac{\int I_s(s)\cos(2\pi L s)\mathrm{d}s}{\int I_s(s)\mathrm{d}s},\qquad S(L) = \frac{\int I_s(s)\sin(2\pi L s)\mathrm{d}s}{\int I_s(s)\mathrm{d}s}\\]

where every integral is evaluated exactly for this piecewise-linear \\(I_s(s)\\).

In the specimen scan, the Kα2 component is an exact replica of the Kα1 component, scaled by the intensity ratio \\(R = I(\mathrm{K}\alpha_2)/I(\mathrm{K}\alpha_1)\\), with \\(0 \le R \lt 1\\), and displaced in \\(s\\) so that its peak lies at the Kα2 Bragg position of the same reflection, that is, at the angle given by Bragg's law for the Kα2 wavelength \\(\lambda_2\\) and the interplanar spacing whose Kα1 peak lies at \\(2\theta_0\\). The specimen profile is therefore the convolution of \\(f\\), of \\(g\\) and of this two-line spectrum. From the computed coefficients of the specimen and instrumental scans, use the convolution theorem to obtain the normalised cosine coefficient \\(A(L)\\) of \\(f\\) at each requested column length; it equals 1 at \\(L = 0\\). The requested column lengths are such that the transform of the instrumental profile does not vanish.

**Uncertainties.** Treat each recorded count of both scans as an independent Poisson variable whose variance equals the recorded count, regard all other inputs as exact, and propagate these variances to first order (linearisation about the recorded counts) through the complete calculation. Report \\(\sigma_A(L)\\) for every requested column length; it is 0 at \\(L = 0\\).

Return the tuple \\((A, \sigma_A)\\) of two arrays; both are checked. \\(A\\) is compared at a relative tolerance of \\(10^{-6}\\) (absolute \\(10^{-9}\\)) and \\(\sigma_A\\) at a relative tolerance of \\(10^{-5}\\) (absolute \\(10^{-12}\\)). Each call must return within about one second on a standard CPU.

Write a function with the following signature:

```python
def subproblem_1(two_theta_meas_deg, intensity_meas, two_theta_inst_deg,
                 intensity_inst, two_theta0_deg, wavelength_ka1_nm,
                 wavelength_ka2_nm, ka2_ratio, L_values):
    """
    Instrument- and doublet-corrected Fourier cosine coefficients of one line.

    Inputs:
        two_theta_meas_deg: 1-D array of 2theta positions in degrees
            (increasing) of the specimen step scan, recorded with
            K-alpha1 + K-alpha2 radiation.
        intensity_meas: 1-D array of specimen counts (intensity per unit
            2theta, including a linear background), same length as
            two_theta_meas_deg.
        two_theta_inst_deg: 1-D array of 2theta positions in degrees
            (increasing) of the instrumental-profile scan, which contains
            K-alpha1 only; its range and step may differ from those of the
            specimen scan.
        intensity_inst: 1-D array of instrumental counts, same length as
            two_theta_inst_deg.
        two_theta0_deg: float, reference angle 2theta_0 in degrees (K-alpha1
            peak position) that defines s = 0 for both scans.
        wavelength_ka1_nm: float, K-alpha1 wavelength in nm.
        wavelength_ka2_nm: float, K-alpha2 wavelength in nm.
        ka2_ratio: float, intensity ratio R = I(K-alpha2)/I(K-alpha1),
            0 <= R < 1.
        L_values: 1-D array of column lengths L in nm (L >= 0) at which the
            coefficients are required; L = 0 need not be included. At every
            requested L the transform of the instrumental profile is non-zero.

    Output:
        Tuple (A, sigma_A) of 1-D numpy arrays, one value per entry of
        L_values and in the same order:
            A: normalised Fourier cosine coefficients of the K-alpha1 physical
                line profile (A = 1 at L = 0).
            sigma_A: standard uncertainty of A from counting statistics
                (Poisson variance of every recorded count equal to the count,
                all counts independent), propagated to first order through
                the whole calculation (sigma_A = 0 at L = 0).
    """
```
````

---

## Subproblem 1 — Background

````markdown
**Why Fourier space.** Every profile recorded on a diffractometer is the specimen's physical line profile, which carries the crystallite-size and lattice-strain information, blurred by the instrument and by the spectrum of the radiation. Stokes showed that the physical profile can be recovered without assuming any analytical shape for any of the functions involved, because convolution becomes multiplication in Fourier space. This deconvolution is the first step of a Warren–Averbach analysis: the size and strain effects are separated later from the Fourier coefficients of the physical profile, and any instrumental or spectral contribution left in them would be misread as microstructure.

**The variable \\(s\\) and the column length \\(L\\).** The natural variable is not the diffraction angle but the magnitude of the diffraction vector, \\(s = 2\sin\theta/\lambda\\), measured from the peak position. It removes the angular distortion of the line shape and makes the Fourier conjugate variable a real length, the column length \\(L\\): the distance between pairs of unit cells measured perpendicular to the diffracting planes. With \\(s\\) in nm\\(^{-1}\\), \\(L\\) is in nm. The mapping between \\(2\theta\\) and \\(s\\) is non-linear, so a uniform step in \\(2\theta\\) becomes a non-uniform grid in \\(s\\), and counts recorded per unit \\(2\theta\\) must be converted into a density per unit \\(s\\) by conservation of the diffracted intensity before they are transformed.

**Oscillatory integrals on coarse grids.** Step scans are usually recorded with a variable step, fine over the peak and coarse in the tails. At long column lengths the kernel \\(\cos(2\pi L s)\\) changes appreciably between neighbouring points, and the trapezoidal rule applied to the product of profile and kernel, which assumes that this product varies linearly, then loses accuracy. Filon's approach avoids the problem: the profile is interpolated between the recorded points, and the oscillating kernel is integrated exactly against the interpolant. Methods that assume a constant step, such as the fast Fourier transform, are not applicable.

**The Kα doublet.** Laboratory radiation contains Kα1 and Kα2 components. The Kα2 line of a reflection is displaced to the angle given by Bragg's law for the longer wavelength, and in the \\(s\\) coordinate computed with the Kα1 wavelength it appears as a weaker, displaced replica of the Kα1 line, so a recorded line is not symmetric about \\(s = 0\\). When the instrumental profile is measured or modelled for Kα1 radiation alone, the Stokes quotient no longer removes the doublet, and the Kα2 contribution must be separated explicitly. Because a displaced replica is a convolution with a two-line spectrum, it can be removed in Fourier space without the point-by-point stripping of the Rachinger method, as proposed by Gangulee.

**Background, truncation, normalisation and noise.** The straight background through the ends of the scan also removes the parts of the tails that lie above it, and the finite scan range removes the remaining tails, so the coefficients are distorted most strongly at small \\(L\\) (the "hook"); they are defined here for the profile as recorded over the supplied range with the stated background. Dividing by the integrated intensity gives \\(A(0) = 1\\) and removes every arbitrary scale factor, such as counting time, detector efficiency and any constant factor in the conversion of units. At large \\(L\\) the deconvolution amplifies counting noise, so coefficients at long column lengths scatter and may even become negative; they are reported as computed.

**Counting statistics.** Every recorded count is a Poisson variable, so its variance equals its expectation, which is estimated by the count itself. All steps up to the normalisation are linear in the counts; the straight background depends on the counts at the two end points, so these two counts influence every background-corrected value. The normalisation and the deconvolution are ratios, and their uncertainty follows from a first-order expansion about the recorded counts. The resulting uncertainty of a coefficient grows, relative to the coefficient, with the column length, because the transforms of both scans decay while the noise does not; this is why deconvolved coefficients at long column lengths scatter, and it is the natural basis for weighting the fits that follow.

References:

Stokes, A. R. (1948). A numerical Fourier-analysis method for the correction of widths and shapes of lines on X-ray powder photographs. Proceedings of the Physical Society, 61(4), 382–391.

Gangulee, A. (1970). Separation of the α1–α2 doublet in X-ray diffraction profiles. Journal of Applied Crystallography, 3(4), 272–277.

Filon, L. N. G. (1928). On a quadrature formula for trigonometric integrals. Proceedings of the Royal Society of Edinburgh, 49, 38–47.

Warren, B. E. (1969). X-ray Diffraction. Addison-Wesley, Chapter 13.
````

---

## Subproblem 1 — Testing Template

```python
import math
import numpy as np
from scipy.special import voigt_profile

CU_KA1, CU_KA2 = 0.1540562, 0.1544390   # Cu K-alpha1 / K-alpha2 wavelengths (nm)
MO_KA1, MO_KA2 = 0.0709300, 0.0713590   # Mo K-alpha1 / K-alpha2 wavelengths (nm)
D_NI_111 = 0.35240 / math.sqrt(3.0)     # Ni (111) spacing (nm)
D_CU_111 = 0.36149 / math.sqrt(3.0)     # Cu (111) spacing (nm)
D_PD_111 = 0.38907 / math.sqrt(3.0)     # Pd (111) spacing (nm)
D_ZNO_002 = 0.52066 / 2.0               # ZnO (002) spacing (nm)


def _bragg_2theta(lam, d):
    return 2.0 * math.degrees(math.asin(lam / (2.0 * d)))


def _grid(center, below, above, step):
    n = int(round((below + above) / step))
    return np.round(center - below + step * np.arange(n + 1), 4)


def _scan(two_theta, tt_ka1, lam1, lam2, d_m, sig, gam, amp, bg0, bg1, seed=None, ka2=0.5):
    """Simulated step scan: Voigt profile in s (nm^-1) with a K-alpha2 replica of
    relative intensity ka2, recorded per unit 2theta (factor cos(theta)) on a
    linear background, with optional counting noise from a fixed seed."""
    tt = np.asarray(two_theta, dtype=float)
    th = np.radians(tt) / 2.0
    s = 2.0 * (np.sin(th) - math.sin(math.radians(tt_ka1) / 2.0)) / lam1
    delta = (lam2 / lam1 - 1.0) / d_m
    dens = voigt_profile(s, sig, gam) + ka2 * voigt_profile(s - delta, sig, gam)
    I = amp * dens * np.cos(th) + bg0 + bg1 * (tt - tt_ka1)
    if seed is not None:
        I = I + np.random.RandomState(seed).normal(0.0, np.sqrt(np.maximum(I, 1.0)))
    return I


def _reflection(m, d1, lam1, lam2, size, strain, meas=(4.0, 4.5, 0.02),
                inst=(1.5, 2.0, 0.01), amp=400.0, bg=(150.0, -2.0), seed=None, ka2=0.5):
    """Specimen scan (K-alpha1 + K-alpha2, ratio ka2) and K-alpha1-only instrumental
    scan of order m. size = (Lorentzian, Gaussian) integral breadths of the size
    profile (nm^-1); strain = first-order (Lorentzian, Gaussian) strain breadths,
    scaled by m^2 and m. Returns (2theta_specimen, counts_specimen,
    2theta_instrument, counts_instrument, reference 2theta_0)."""
    d_m = d1 / m
    tt1 = _bragg_2theta(lam1, d_m)
    sig_f2 = (size[1] ** 2 + (m * strain[1]) ** 2) / (2.0 * math.pi)
    gam_f = (size[0] + m ** 2 * strain[0]) / math.pi
    sig_i, gam_i = 0.0012 + 0.0004 * m, 0.0010 + 0.0003 * m
    sig_h, gam_h = math.sqrt(sig_f2 + sig_i ** 2), gam_f + gam_i
    centre = round(tt1, 2)
    tt_h = _grid(centre, *meas)
    tt_g = _grid(centre, *inst)
    I_h = _scan(tt_h, tt1, lam1, lam2, d_m, sig_h, gam_h, amp, bg[0], bg[1], seed, ka2)
    I_g = _scan(tt_g, tt1, lam1, lam2, d_m, sig_i, gam_i, 100.0, 20.0, 0.0,
                None if seed is None else seed + 1000, 0.0)
    return tt_h, I_h, tt_g, I_g, round(tt1, 3)


def _check(out, expected):
    assert len(out) == 2, f"expected (A, sigma_A), got {len(out)} outputs"
    for name, got, exp, rtol, atol in (("A", out[0], expected[0], 1e-6, 1e-9),
                                       ("sigma_A", out[1], expected[1], 1e-5, 1e-12)):
        got = np.asarray(got, dtype=float).ravel()
        assert got.shape == (len(exp),), f"{name}: expected {len(exp)} values, got shape {got.shape}"
        assert np.all(np.isfinite(got)), f"{name} must be finite"
        for i, (a, b) in enumerate(zip(got, exp)):
            assert math.isclose(a, b, rel_tol=rtol, abs_tol=atol), f"{name}[{i}]: expected {b}, got {a}"


def test_case_1():
    # Ni (111), Cu K-alpha doublet in the specimen scan, K-alpha1-only instrumental
    # scan, noise-free: the K-alpha2 replica must be removed in Fourier space.
    tt_h, I_h, tt_g, I_g, tt0 = _reflection(1, D_NI_111, CU_KA1, CU_KA2,
                                            (1 / 28, 0.028), (0.0012, 0.010))
    out = subproblem_1(tt_h, I_h, tt_g, I_g, tt0, CU_KA1, CU_KA2, 0.5,
                     np.array([0.0, 2.0, 5.0, 10.0, 20.0, 30.0]))
    _check(out, (
        [
          1.0, 0.877878001, 0.6635645135,
          0.3722167839, 0.07708808237, 0.009132122198],
        [
          0.0, 0.01693609705, 0.01301458984,
          0.007532881085, 0.002828899356, 0.004098975548]))


def test_case_2():
    # Ni (222) at 2theta ~ 98 deg: large doublet separation, strongly varying
    # cos(theta) across the scan and coarse 0.02 deg steps at long column lengths.
    tt_h, I_h, tt_g, I_g, tt0 = _reflection(2, D_NI_111, CU_KA1, CU_KA2,
                                            (1 / 28, 0.028), (0.0012, 0.010))
    out = subproblem_1(tt_h, I_h, tt_g, I_g, tt0, CU_KA1, CU_KA2, 0.5,
                     np.array([0.0, 2.0, 5.0, 10.0, 20.0, 30.0]))
    _check(out, (
        [
          1.0, 0.8815458969, 0.6353926579,
          0.3202308858, 0.04659888183, 0.003191824949],
        [
          0.0, 0.01735122801, 0.01467292296,
          0.006255798294, 0.005987315393, 0.003228269329]))


def test_case_3():
    # Pd (111) with Mo radiation (ratio 0.52) at low angle, counting noise, and
    # L = 0 absent: the normalisation must use the separately integrated intensity.
    tt_h, I_h, tt_g, I_g, tt0 = _reflection(1, D_PD_111, MO_KA1, MO_KA2,
                                            (1 / 36, 0.020), (0.0008, 0.008),
                                            meas=(2.0, 2.5, 0.01), inst=(1.0, 1.5, 0.005),
                                            seed=7, ka2=0.52)
    out = subproblem_1(tt_h, I_h, tt_g, I_g, tt0, MO_KA1, MO_KA2, 0.52,
                     np.array([1.0, 4.0, 8.0, 15.0, 25.0]))
    _check(out, (
        [
          0.9771346886, 0.7966796369, 0.5910063255,
          0.3064391095, 0.09611265462],
        [
          0.02085951899, 0.02123098412, 0.01659468997,
          0.009160324138, 0.004264384888]))


def test_case_4():
    # Monochromated specimen scan (ratio 0): no doublet to remove, but the
    # conversion to s, the exact transform and the deconvolution are still required.
    tt_h, I_h, tt_g, I_g, tt0 = _reflection(1, D_NI_111, CU_KA1, CU_KA2,
                                            (1 / 28, 0.028), (0.0012, 0.010), ka2=0.0)
    out = subproblem_1(tt_h, I_h, tt_g, I_g, tt0, CU_KA1, CU_KA2, 0.0,
                     np.array([0.0, 3.0, 6.0, 12.0, 24.0]))
    _check(out, (
        [
          1.0, 0.8040076275, 0.5979655863,
          0.2840584099, 0.03511688515],
        [
          0.0, 0.01943397442, 0.01600093662,
          0.007731838532, 0.002581003625]))


def test_case_5():
    # ZnO (002): specimen recorded with a variable step (0.01 deg over the peak,
    # 0.05 deg in the tails), instrumental scan on a different grid.
    tt1 = _bragg_2theta(CU_KA1, D_ZNO_002)
    c = round(tt1, 2)
    tt_h = np.round(np.concatenate([c - 4.0 + 0.05 * np.arange(64),
                                    c - 0.8 + 0.01 * np.arange(161),
                                    c + 0.85 + 0.05 * np.arange(74)]), 4)
    sig_i, gam_i = 0.0016, 0.0013
    sig_h = math.sqrt((0.020 ** 2 + 0.012 ** 2) / (2.0 * math.pi) + sig_i ** 2)
    gam_h = (1 / 24 + 0.0015) / math.pi + gam_i
    I_h = _scan(tt_h, tt1, CU_KA1, CU_KA2, D_ZNO_002, sig_h, gam_h, 400.0, 120.0, -1.0, ka2=0.5)
    tt_g = _grid(c, 1.5, 2.0, 0.01)
    I_g = _scan(tt_g, tt1, CU_KA1, CU_KA2, D_ZNO_002, sig_i, gam_i, 100.0, 20.0, 0.0, ka2=0.0)
    out = subproblem_1(tt_h, I_h, tt_g, I_g, round(tt1, 3), CU_KA1, CU_KA2, 0.5,
                     np.array([0.0, 2.0, 6.0, 12.0, 24.0]))
    _check(out, (
        [
          1.0, 0.8637122643, 0.5793569551,
          0.286937506, 0.0486742471],
        [
          0.0, 0.01547676454, 0.01119923231,
          0.005769382271, 0.002436655525]))


def test_case_6():
    # ZnO (004) with a steeply sloping background and counting noise
    tt_h, I_h, tt_g, I_g, tt0 = _reflection(2, D_ZNO_002, CU_KA1, CU_KA2,
                                            (1 / 24, 0.020), (0.0015, 0.012),
                                            bg=(300.0, -15.0), seed=5)
    out = subproblem_1(tt_h, I_h, tt_g, I_g, tt0, CU_KA1, CU_KA2, 0.5,
                     np.array([0.0, 1.5, 4.5, 9.0, 18.0, 27.0]))
    _check(out, (
        [
          1.0, 0.9057027038, 0.6475082985,
          0.3506741532, 0.07538105013, 0.009726273651],
        [
          0.0, 0.02224420113, 0.01824906312,
          0.008545644479, 0.004463653175, 0.007204595384]))
```

---

## Subproblem 2 — Prompt

````markdown
From the size Fourier coefficients \\(A_S(L)\\) of a nanocrystalline powder, determine eight quantities: the area-weighted and volume-weighted mean column lengths, the median and the logarithmic standard deviation of the log-normal distribution of spherical crystallite diameters that is consistent with them, the size coefficient and the column-length distribution predicted by that distribution, and the integral breadth and the full width at half maximum of the size-broadened line profile that it predicts. The function returns all eight, in this order.

The input is \\(A_S(L)\\) on a grid of column lengths \\(0 = L_0 \lt L_1 \lt \dots\\) in nm, with \\(A_S(0) = 1\\); entries may be 0 where the coefficient was lost in the noise, and they are used as supplied. Two zero-based indices \\(i_{\mathrm{lo}} \lt i_{\mathrm{hi}}\\) are also given. Diffraction by a crystallite is the sum of the contributions of columns of unit cells parallel to the diffraction vector. If \\(p(l)\\) is the distribution of the lengths \\(l\\) of all columns in the sample, every column of the same cross-section being counted once, then

\\[A_S(L) = \frac{1}{\langle l\rangle}\int_L^\infty (l - L)p(l)\mathrm{d}l,\qquad \langle L\rangle_{\mathrm{area}} = \langle l\rangle,\qquad \langle L\rangle_{\mathrm{vol}} = \frac{\langle l^2\rangle}{\langle l\rangle}\\]

Carry out the following steps.

1. **Area-weighted mean column length.** Fit an ordinary (unweighted) least-squares straight line \\(A_S \approx c_0 + c_1 L\\) to the points with \\(i_{\mathrm{lo}} \le i \le i_{\mathrm{hi}}\\) (both ends included), which avoid the hook at small \\(L\\), and take \\(\langle L\rangle_{\mathrm{area}}\\) as the column length at which the line reaches \\(A_S = 0\\).
2. **Volume-weighted mean column length.** \\(\langle L\rangle_{\mathrm{vol}} = 2\int A_S(L)\mathrm{d}L\\), evaluated by the trapezoidal rule over the whole supplied grid, including points where \\(A_S = 0\\).
3. **Log-normal spheres.** Assume that the crystallites are spheres whose diameters \\(D\\) follow a log-normal number distribution with median \\(D_{\mathrm{med}}\\) and logarithmic standard deviation \\(\sigma \ge 0\\) (the standard deviation of \\(\ln D\\)). Determine \\(\sigma\\) and \\(D_{\mathrm{med}}\\) so that the area-weighted and volume-weighted mean column lengths of this powder of spheres equal the two values obtained above. If the ratio \\(\langle L\rangle_{\mathrm{vol}}/\langle L\rangle_{\mathrm{area}}\\) does not exceed the value it takes for identical spheres, set \\(\sigma = 0\\); in every case, obtain \\(D_{\mathrm{med}}\\) from the condition on \\(\langle L\rangle_{\mathrm{area}}\\).
4. **Model size coefficient.** Evaluate \\(A_{\mathrm{model}}(L_i)\\), the size coefficient of this powder of spheres, at every supplied column length.
5. **Column-length distribution.** Evaluate \\(p(L_i)\\) in nm\\(^{-1}\\), the distribution of the lengths of all columns in this powder of spheres (normalised to unit integral; every column of the same cross-section is counted once, so that each sphere contributes columns in proportion to its projected area), at every supplied column length.
6. **Predicted line profile.** The size-broadened line profile of this powder of spheres, an intensity per unit \\(s\\) in nm\\(^{-1}\\), is \\(I(s) = \int_{-\infty}^{\infty} A_{\mathrm{model}}(|L|)\cos(2\pi L s)\mathrm{d}L\\), where \\(A_{\mathrm{model}}(L)\\) is the size coefficient of step 4 as a function of the continuous column length \\(L \ge 0\\). Evaluate its integral breadth \\(\beta = \int I(s)\mathrm{d}s / I(0)\\) and its full width at half maximum \\(w = 2s_{1/2}\\), where \\(s_{1/2}\\) is the smallest positive \\(s\\) at which \\(I(s) = I(0)/2\\), both in nm\\(^{-1}\\).

For \\(\sigma = 0\\) the powder consists of identical spheres of diameter \\(D_{\mathrm{med}}\\).

Raise a ValueError if L_values and A_size are not one-dimensional arrays of the same length (at least two) with finite entries, if L_values does not start at 0 or does not increase strictly, if A_size[0] is not 1, if fit_index_lo and fit_index_hi are not integers with \\(0 \le i_{\mathrm{lo}} \lt i_{\mathrm{hi}} \lt\\) len(L_values), or if the straight line of step 1 does not reach \\(A_S = 0\\) at a positive column length.

\\(\langle L\rangle_{\mathrm{area}}\\), \\(\langle L\rangle_{\mathrm{vol}}\\), \\(D_{\mathrm{med}}\\), \\(\beta\\) and \\(w\\) are compared at a relative tolerance of \\(10^{-6}\\), \\(\sigma\\) at a relative tolerance of \\(10^{-6}\\) or an absolute tolerance of \\(10^{-9}\\), and \\(A_{\mathrm{model}}\\) and \\(p\\) at a relative tolerance of \\(10^{-6}\\) or absolute tolerances of \\(10^{-10}\\) and \\(10^{-12}\\) nm\\(^{-1}\\). Each call must return within about one second on a standard CPU.

Write a function with the following signature:

```python
def subproblem_2(L_values, A_size, fit_index_lo, fit_index_hi):
    """
    Mean column lengths, log-normal sphere-size distribution and predicted
    line-profile widths from A_S(L).

    Inputs:
        L_values: 1-D array of column lengths L in nm, increasing, with
            L_values[0] = 0.
        A_size: 1-D array of size Fourier coefficients A_S(L) at L_values
            (A_size[0] = 1); entries may be 0 where the coefficient was lost
            in the noise.
        fit_index_lo: int, zero-based index of the first point of the linear
            fit for the area-weighted mean column length.
        fit_index_hi: int, zero-based index of the last point of that fit
            (inclusive).

    Output:
        Tuple (L_area, L_vol, D_median, sigma, A_model, p_col, beta, fwhm):
            L_area: float, area-weighted mean column length in nm.
            L_vol: float, volume-weighted mean column length in nm.
            D_median: float, median of the log-normal number distribution of
                sphere diameters in nm.
            sigma: float, logarithmic standard deviation of that distribution
                (dimensionless, >= 0).
            A_model: 1-D numpy array, size Fourier coefficient of a powder of
                spheres with that diameter distribution, at L_values.
            p_col: 1-D numpy array, column-length distribution p(L) of that
                powder in nm^-1 (unit integral), at L_values.
            beta: float, integral breadth of the size-broadened line profile
                of that powder, in nm^-1.
            fwhm: float, full width at half maximum of that profile, in nm^-1.

    Raises:
        ValueError under the conditions stated above.
    """
```
````

---

## Subproblem 2 — Background

````markdown
**What a Warren–Averbach analysis measures.** It does not yield "the crystallite size". It yields the size Fourier coefficient \\(A_S(L)\\), which is determined by the distribution of column lengths in the sample, and two different averages of that distribution follow from it. The initial slope of \\(A_S\\) is \\(-1/\langle l\rangle\\), so the tangent at the origin meets the \\(L\\) axis at the area-weighted mean column length; the integral of \\(A_S\\) is \\(\langle l^2\rangle/(2\langle l\rangle)\\), so twice the integral is the volume-weighted mean column length. By the Cauchy–Schwarz inequality \\(\langle L\rangle_{\mathrm{vol}} \ge \langle L\rangle_{\mathrm{area}}\\), with equality only when all columns have the same length. Because the integral is evaluated over the supplied, finite grid, a grid that ends before \\(A_S\\) has decayed biases \\(\langle L\rangle_{\mathrm{vol}}\\) downward; the value computed is the one supported by the supplied grid.

**The hook effect and the tangent construction.** The second derivative of \\(A_S\\) equals \\(p(L)/\langle l\rangle \ge 0\\), so the size coefficient is convex, and its initial slope is its most delicate feature. Background errors, truncation of the scans and peak overlap distort the smallest-\\(L\\) coefficients and bend the curve near the origin. The standard remedy is to discard the affected points, fit a straight line to the adjacent, nearly linear region and extrapolate it to \\(A_S = 0\\); where the linear region lies is judged from the data, so it is supplied as an index range. The extrapolated line does not in general pass through \\(A_S = 1\\) at \\(L = 0\\), so its zero is not \\(-1/c_1\\).

**From column lengths to grain diameters.** Microscopy measures grain diameters, and the properties of a nanocrystalline material depend on them, so the two column-length averages must be converted into a diameter distribution. This requires a shape and a form for the distribution. Every crystallite contributes columns of many lengths: a sphere of diameter \\(D\\) contains columns from zero length at its rim to \\(D\\) through its centre, in numbers proportional to the projected area they occupy. Averaging over a powder therefore weights each sphere by the number of columns it contributes, and the resulting averages are ratios of moments of the diameter distribution. Nanocrystalline materials prepared by inert-gas condensation, milling or electrodeposition usually show approximately log-normal grain-size distributions, for which every moment \\(\langle D^k\rangle\\), and every partial moment over diameters above a given value, has a closed form. Krill and Birringer showed that the two weighted averages obtained from \\(A_S\\) are sufficient to estimate both parameters of such a distribution, even though the distribution cannot be obtained directly from the Fourier coefficients.

**The monodisperse limit and the model curve.** The ratio \\(\langle L\rangle_{\mathrm{vol}}/\langle L\rangle_{\mathrm{area}}\\) depends only on the width of the distribution and takes its smallest value for identical spheres. A measured ratio at or below that value cannot be represented by a log-normal distribution of spheres; it indicates a distribution too narrow to be resolved, given the biases of the finite \\(L\\) range and of the tangent construction, and it is reported as \\(\sigma = 0\\). The size coefficient predicted by the fitted distribution is the most direct check of the result: it can be compared point by point with the measured \\(A_S(L)\\), and systematic deviations reveal a wrong shape assumption, an unsuitable fit window or strain contributions left in the size coefficients. For identical spheres the predicted coefficient reaches zero at \\(L = D\\), because no column is longer than a diameter. The column-length distribution of the model is related to its size coefficient through \\(p(L) = \langle l\rangle\mathrm{d}^2A_S/\mathrm{d}L^2\\) and shows directly which column lengths dominate the diffraction; for a powder it is never simply the distribution of diameters, because every crystallite contributes columns of all lengths up to its own size.

**Line-profile widths.** A size distribution is more often characterised through the widths of the diffraction line than through its Fourier coefficients. The size-broadened profile is the Fourier transform of the size coefficient; its area equals the coefficient at \\(L = 0\\) and its peak value equals twice the integral of the coefficient, so its integral breadth is the reciprocal of the volume-weighted mean column length of the coefficients from which it is built (the Stokes–Wilson relation). The full width at half maximum obeys no such relation. It depends on the shape of the profile, which for a powder of spheres is the volume-weighted superposition of the profiles of the individual spheres, each obtained from the size coefficient of a single sphere. For identical spheres the ratio of the full width at half maximum to the integral breadth lies between its Gaussian (0.94) and Lorentzian (0.64) values, and it decreases as the distribution broadens, because the largest crystallites add a narrow component to the top of the profile while the smallest ones add broad tails. The two widths therefore carry independent information on the distribution, and comparing the predicted widths with those of the measured physical profile checks the fitted distribution against quantities that do not depend on the fit window.

References:

Krill, C. E., & Birringer, R. (1998). Estimating grain-size distributions in nanocrystalline materials from X-ray diffraction profile analysis. Philosophical Magazine A, 77(3), 621–640.

Langford, J. I., Louër, D., & Scardi, P. (2000). Effect of a crystallite size distribution on X-ray diffraction line profiles and whole-powder-pattern fitting. Journal of Applied Crystallography, 33(3), 964–974.

Warren, B. E. (1969). X-ray Diffraction. Addison-Wesley, Chapter 13.
````

---

## Subproblem 2 — Testing Template

```python
import math
import numpy as np


def _check(out, expected, rtol=1e-6):
    assert len(out) == 8, f"expected 8 outputs, got {len(out)}"
    L_area, L_vol, D_med, sigma, A_mod, p_col, beta, fwhm = out
    e_area, e_vol, e_med, e_sigma, e_mod, e_p, e_beta, e_fwhm = expected
    assert math.isclose(L_area, e_area, rel_tol=rtol), f"L_area: expected {e_area} nm, got {L_area}"
    assert math.isclose(L_vol, e_vol, rel_tol=rtol), f"L_vol: expected {e_vol} nm, got {L_vol}"
    assert math.isclose(D_med, e_med, rel_tol=rtol), f"D_median: expected {e_med} nm, got {D_med}"
    assert math.isclose(sigma, e_sigma, rel_tol=rtol, abs_tol=1e-9), f"sigma: expected {e_sigma}, got {sigma}"
    A_mod = np.asarray(A_mod, dtype=float).ravel()
    assert A_mod.shape == (len(e_mod),), f"A_model: expected {len(e_mod)} values, got shape {A_mod.shape}"
    assert np.all(np.isfinite(A_mod)), "A_model must be finite"
    for i, (a, b) in enumerate(zip(A_mod, e_mod)):
        assert math.isclose(a, b, rel_tol=rtol, abs_tol=1e-10), f"A_model[{i}]: expected {b}, got {a}"
    p_col = np.asarray(p_col, dtype=float).ravel()
    assert p_col.shape == (len(e_p),), f"p_col: expected {len(e_p)} values, got shape {p_col.shape}"
    assert np.all(np.isfinite(p_col)), "p_col must be finite"
    for i, (a, b) in enumerate(zip(p_col, e_p)):
        assert math.isclose(a, b, rel_tol=rtol, abs_tol=1e-12), f"p_col[{i}]: expected {b}, got {a}"
    assert math.isclose(beta, e_beta, rel_tol=rtol), f"beta: expected {e_beta} nm^-1, got {beta}"
    assert math.isclose(fwhm, e_fwhm, rel_tol=rtol), f"fwhm: expected {e_fwhm} nm^-1, got {fwhm}"


def _raises(*args):
    try:
        subproblem_2(*args)
    except ValueError:
        return True
    return False


def test_case_1():
    # Size coefficients of nanocrystalline Ni from a noise-free (111)/(222) analysis;
    # the small-L hook is excluded by the fit window
    L = 1.5 * np.arange(25)
    A_S = np.array([1.0, 0.91574527, 0.80808378, 0.70575339, 0.61049445, 0.52127588,
          0.44043705, 0.3681013, 0.304204, 0.24863446, 0.20088737, 0.16059137,
          0.12689036, 0.09921723, 0.07665699, 0.05860739, 0.044333, 0.03305852,
          0.02455242, 0.01784645, 0.01296431, 0.00924136, 0.00655627, 0.00455518,
          0.00317277])
    out = subproblem_2(L, A_S, 2, 6)
    _check(out, (
        16.06569456, 18.90082255, 21.54860071, 0.2115092926,
        [
          1.0, 0.9067712529, 0.8143698929, 0.7236233073, 0.635358883,
          0.5504040073, 0.4695860618, 0.3937322368, 0.3236670793, 0.2601971773,
          0.20405857, 0.1558077244, 0.1156797639, 0.08348240001, 0.0585880823,
          0.04003296978, 0.02667845603, 0.01737459624, 0.01108206985, 0.006937932943,
          0.004272342963, 0.002592999037, 0.001553993676, 0.0009211888526, 0.0005409701427],
        [
          0.0, 0.005907799587, 0.01181559917, 0.01772339876, 0.02363119833,
          0.02953898901, 0.03544614865, 0.04134061625, 0.04712900984, 0.05245122088,
          0.05651518767, 0.05826285436, 0.05689513848, 0.05235125894, 0.04536773963,
          0.0371369292, 0.02885234604, 0.02139289913, 0.01522271598, 0.01045047027,
          0.006954788637, 0.00450602983, 0.002853002939, 0.001771083278, 0.001081061606],
        0.05290775029, 0.04239345848))


def test_case_2():
    # Pd from three orders with counting noise (2 nm grid)
    L = 2.0 * np.arange(21)
    A_S = np.array([1.0, 0.93344643, 0.82421619, 0.7208538, 0.62134685, 0.53196525,
          0.45364547, 0.38246899, 0.31611386, 0.26217225, 0.21984041, 0.17452011,
          0.14314601, 0.11716655, 0.09285473, 0.07072767, 0.06004776, 0.04011635,
          0.03494114, 0.02044138, 0.02112281])
    out = subproblem_2(L, A_S, 2, 5)
    _check(out, (
        20.82000044, 26.12237042, 23.77517249, 0.3302952187,
        [
          1.0, 0.9041206948, 0.8093344226, 0.7167342145, 0.6274129382,
          0.5424602166, 0.4629423005, 0.3898408598, 0.3239489207, 0.2657626807,
          0.215418401, 0.1726972149, 0.1370876981, 0.1078788688, 0.08425650277,
          0.0653848093, 0.05046577724, 0.03877597386, 0.0296845872, 0.02265782351,
          0.01725451258],
        [
          0.0, 0.005689236463, 0.01137847254, 0.01706744789, 0.02274585239,
          0.02832187563, 0.033479242, 0.03765712672, 0.04026856128, 0.0409749381,
          0.03980684053, 0.0371028899, 0.0333594136, 0.02908553408, 0.02470952514,
          0.02054015953, 0.01676717614, 0.01348217898, 0.01070591481, 0.008413813884,
          0.006556312338],
        0.03828136513, 0.02933486531))


def test_case_3():
    # ZnO nanoparticles (1.25 nm grid, fit window 3..7): a broad distribution
    L = 1.25 * np.arange(25)
    A_S = np.array([1.0, 0.93553475, 0.84703211, 0.76283051, 0.67646453, 0.5958707,
          0.52689635, 0.46184325, 0.4074665, 0.35342549, 0.30833571, 0.26481511,
          0.23015368, 0.19825139, 0.16847619, 0.14504836, 0.12672459, 0.11094215,
          0.08522562, 0.08166959, 0.10736093, 0.06687182, 0.09644084, 0.03619636,
          0.03253714])
    out = subproblem_2(L, A_S, 3, 7)
    _check(out, (
        16.30899379, 20.27536275, 19.05662852, 0.3160802651,
        [
          1.0, 0.9234451878, 0.8474304614, 0.7724959069, 0.6991816088,
          0.6280275966, 0.5595731472, 0.4943527667, 0.4328834355, 0.3756388688,
          0.3230133226, 0.2752845793, 0.2325874074, 0.1949046516, 0.1620767861,
          0.1338257638, 0.109786771, 0.08954169736, 0.0726497111, 0.05867227534,
          0.04719163949, 0.03782303029, 0.03022144669, 0.02408423597, 0.01915063391],
        [
          0.0, 0.005637284924, 0.01127456985, 0.01691185249, 0.02254887974,
          0.02818050259, 0.03377000944, 0.03918880127, 0.04416605099, 0.04831482606,
          0.05123820487, 0.05265269831, 0.05246478659, 0.05077815881, 0.04784761042,
          0.04401012035, 0.03961908935, 0.03499593247, 0.03040254397, 0.02603167369,
          0.02200973568, 0.01840659781, 0.01524810404, 0.01252854439, 0.01022155069],
        0.04932094248, 0.03802345598))


def test_case_4():
    # Identical spheres of 12 nm: the ratio <L>_vol/<L>_area does not exceed its
    # monodisperse value, so sigma = 0, and the model vanishes for L >= D_median;
    # the predicted profile is that of identical spheres, so beta != 1/L_vol
    L = 1.0 * np.arange(17)
    A_S = np.array([1.0, 0.87528935, 0.75231481, 0.6328125, 0.51851852, 0.41116898,
          0.3125, 0.22424769, 0.14814815, 0.0859375, 0.03935185, 0.01012731,
          0.0, 0.0, 0.0, 0.0, 0.0])
    out = subproblem_2(L, A_S, 2, 5)
    _check(out, (
        8.586470031, 9.02083332, 12.87970505, 0.0,
        [
          1.0, 0.8837717253, 0.7689475682, 0.6569316465, 0.5491280777,
          0.4469409796, 0.3517744698, 0.265032666, 0.1881196858, 0.1224396469,
          0.06939666698, 0.03039486366, 0.006838354623, 0.0, 0.0,
          0.0, 0.0],
        [
          0.0, 0.01205641422, 0.02411282845, 0.03616924267, 0.0482256569,
          0.06028207112, 0.07233848534, 0.08439489957, 0.09645131379, 0.108507728,
          0.1205641422, 0.1326205565, 0.1446769707, 0.0, 0.0,
          0.0, 0.0],
        0.1035220394, 0.08592232901))


def test_case_5():
    # Small Cu crystallites with counting noise: A_S = 0 where the coefficients were
    # lost in the noise, and these points still enter the L_vol integral
    L = 1.0 * np.arange(31)
    A_S = np.array([1.0, 0.93255083, 0.80512767, 0.70638712, 0.61045013, 0.51899047,
          0.44585341, 0.3797432, 0.32181712, 0.26948871, 0.22900301, 0.18692172,
          0.15513584, 0.13164894, 0.10609547, 0.08533575, 0.06973087, 0.05492798,
          0.0495009, 0.0427459, 0.03804753, 0.0620852, 0.0, 0.01955905,
          0.01833243, 0.0, 0.0, 0.0, 0.0, 0.0,
          0.0])
    out = subproblem_2(L, A_S, 3, 7)
    _check(out, (
        11.5080684, 13.4789585, 15.60774804, 0.2007446509,
        [
          1.0, 0.9132141408, 0.827086461, 0.74227514, 0.6594383574,
          0.5792342926, 0.5023211248, 0.4293570221, 0.3609998932, 0.2979048898,
          0.2407111736, 0.1900004573, 0.1462142582, 0.1095455758, 0.07985218298,
          0.05663889848, 0.03912203966, 0.02634979362, 0.01733436341, 0.01115911037,
          0.007043617976, 0.00436776083, 0.002665896977, 0.001604463082, 0.0009537727279,
          0.000560864988, 0.0003267242677, 0.0001887856692, 0.0001083234989, 6.178656087e-05,
          3.506621602e-05],
        [
          0.0, 0.00757437401, 0.01514874802, 0.02272312203, 0.03029749604,
          0.03787186978, 0.04544620057, 0.05301889876, 0.06056860745, 0.06796154216,
          0.07473716758, 0.07992889929, 0.08224001547, 0.08062202795, 0.07485772669,
          0.06571738799, 0.05463217906, 0.04315967605, 0.03254887123, 0.02354629576,
          0.01641721132, 0.01108146409, 0.007270702099, 0.004653892551, 0.002915547683,
          0.001792779412, 0.001084744036, 0.0006472619416, 0.0003816167005, 0.0002226942642,
          0.0001288176837],
        0.07418970835, 0.05964340751))


def test_case_6():
    # Exact size coefficient of log-normal spheres (median 15 nm, sigma 0.35) on a
    # long, fine grid: the curvature of A_S biases the tangent construction
    L = 1.0 * np.arange(61)
    A_S = np.array([1.0, 0.92646511, 0.85344243, 0.78144415, 0.71098247, 0.64256932,
          0.57671427, 0.51391659, 0.45464686, 0.39931882, 0.34825813, 0.30167707,
          0.25966181, 0.22217403, 0.18906471, 0.16009578, 0.13496512, 0.11333119,
          0.09483476, 0.07911654, 0.06583037, 0.05465224, 0.04528573, 0.03746467,
          0.03095367, 0.02554716, 0.02106747, 0.01736243, 0.01430266, 0.01177884,
          0.0096991, 0.00798657, 0.00657718, 0.0054177, 0.00446401, 0.00367966,
          0.00303453, 0.00250385, 0.00206718, 0.00170776, 0.0014118, 0.00116797,
          0.00096698, 0.00080121, 0.0006644, 0.0005514, 0.00045802, 0.00038078,
          0.00031685, 0.00026389, 0.00021997, 0.00018354, 0.00015328, 0.00012812,
          0.0001072, 0.00008977, 0.00007525, 0.00006314, 0.00005302, 0.00004457,
          0.0000375])
    out = subproblem_2(L, A_S, 1, 4)
    _check(out, (
        13.88684483, 17.2843637, 16.17899775, 0.3179264816,
        [
          1.0, 0.9280643185, 0.8565781359, 0.7859909511, 0.7167522623,
          0.6493115449, 0.5841179307, 0.5216181937, 0.4622497556, 0.4064251811,
          0.3545079943, 0.3067845994, 0.2634398151, 0.2245424557, 0.1900438588,
          0.159788482, 0.1335331931, 0.1109710104, 0.09175543744, 0.07552257979,
          0.06190941386, 0.05056759663, 0.04117292915, 0.03343101957, 0.02707988454,
          0.02189025755, 0.0176643035, 0.01423332353, 0.01145490558, 0.00920985563,
          0.007399140996, 0.005940994536, 0.00476826611, 0.003826062463, 0.003069685937,
          0.002462862546, 0.001976238183, 0.001586115689, 0.00127340337, 0.001022745858,
          0.0008218100391, 0.0006607013573, 0.0005314887449, 0.0004278193544, 0.0003446070728,
          0.0002777813315, 0.0002240849752, 0.0001809118978, 0.0001461768105, 0.0001182109107,
          9.567837638e-05, 7.750958064e-05, 6.28477065e-05, 5.100609024e-05, 4.143414505e-05,
          3.369014212e-05, 2.741946897e-05, 2.233726092e-05, 1.821452319e-05, 1.486703913e-05,
          1.214650185e-05],
        [
          0.0, 0.006242121023, 0.01248424205, 0.01872636199, 0.02496834607,
          0.03120715403, 0.03741886795, 0.04351114389, 0.04926916169, 0.05435112203,
          0.05835774846, 0.06094142562, 0.06189857474, 0.06120997948, 0.05902697397,
          0.05562255232, 0.05133094785, 0.04649336565, 0.04141879002, 0.03636158255,
          0.03151333554, 0.0270048, 0.02291373609, 0.01927536644, 0.01609315578,
          0.01334857666, 0.01100923021, 0.009035162681, 0.007383493648, 0.006011606004,
          0.004879189458, 0.003949418429, 0.003189508033, 0.002570845715, 0.002068850458,
          0.001662671428, 0.00133480496, 0.001070683135, 0.000858268063, 0.0006876721214,
          0.0005508148415, 0.0004411207134, 0.0003532580962, 0.0002829169909, 0.0002266221246,
          0.0001815772405, 0.0001455363964, 0.0001166982783, 9.361987784e-05, 7.514630482e-05,
          6.035393133e-05, 4.850448203e-05, 3.900805985e-05, 3.139343377e-05, 2.528420567e-05,
          2.037972165e-05, 1.643980187e-05, 1.327253702e-05, 1.072454285e-05, 8.673182181e-06,
          7.020359703e-06],
        0.05785576012, 0.04456874208))


def test_case_7():
    # Invalid inputs
    L = 1.0 * np.arange(12)
    A_S = np.array([1.0, 0.9, 0.8, 0.7, 0.6, 0.5, 0.41, 0.33, 0.26, 0.2, 0.15, 0.11])
    assert not _raises(L, A_S, 1, 4)
    assert not _raises(L, A_S, np.int64(1), np.int64(4))
    assert _raises(L, A_S, -1, 4)                    # negative index
    assert _raises(L, A_S, 4, 4)                     # degenerate window
    assert _raises(L, A_S, 5, 3)                     # reversed window
    assert _raises(L, A_S, 2, 12)                    # window beyond the grid
    assert _raises(L, A_S, 1.5, 4)                   # non-integer index
    assert _raises(L[:-1], A_S, 1, 4)                # length mismatch
    assert _raises(L + 0.5, A_S, 1, 4)               # grid does not start at L = 0
    L_bad = L.copy(); L_bad[5] = L_bad[4]
    assert _raises(L_bad, A_S, 1, 4)                 # grid not strictly increasing
    A_bad = A_S.copy(); A_bad[0] = 0.98
    assert _raises(L, A_bad, 1, 4)                   # A_S(0) != 1
    A_bad = A_S.copy(); A_bad[7] = np.nan
    assert _raises(L, A_bad, 1, 4)                   # non-finite coefficient
    # window placed on the noise floor: the fitted line rises and never
    # reaches A_S = 0 at a positive column length
    A_flat = np.array([1.0, 0.6, 0.3, 0.1, 0.0, 0.0, 0.0, 0.02, 0.0, 0.03, 0.0, 0.05])
    assert _raises(L, A_flat, 6, 11)


def test_case_8():
    # Broad distribution of small crystallites (exact size coefficient of log-normal
    # spheres, median 8 nm, sigma 0.6): a strongly convex A_S and a predicted line
    # profile with heavy tails, far from Gaussian or Lorentzian
    L = 1.0 * np.arange(61)
    A_S = np.array([1.0, 0.92396145, 0.84908069, 0.77648408, 0.70716784, 0.64189081,
          0.5811334, 0.52511904, 0.47386562, 0.42724239, 0.38502037, 0.34691228,
          0.312602, 0.28176525, 0.25408337, 0.229252, 0.20698614, 0.18702267,
          0.16912115, 0.15306359, 0.13865345, 0.1257143, 0.11408824, 0.10363433,
          0.09422694, 0.08575421, 0.07811667, 0.07122583, 0.06500304, 0.05937835,
          0.05428957, 0.04968135, 0.04550444, 0.04171495, 0.0382738, 0.03514611,
          0.03230075, 0.02970992, 0.02734877, 0.02519506, 0.02322888, 0.02143236,
          0.01978951, 0.01828594, 0.01690872, 0.01564624, 0.01448803, 0.01342466,
          0.01244761, 0.01154922, 0.01072255, 0.00996133, 0.00925987, 0.00861303,
          0.00801616, 0.00746502, 0.00695578, 0.00648494, 0.00604933, 0.00564606,
          0.0052725])
    out = subproblem_2(L, A_S, 1, 4)
    _check(out, (
        13.76139586, 20.99149142, 9.642353153, 0.5517851745,
        [
          1.0, 0.9274746606, 0.8557993717, 0.7858180213, 0.718334841,
          0.6540503672, 0.5935039541, 0.5370493986, 0.4848625151, 0.436967907,
          0.393272897, 0.3536008808, 0.3177202625, 0.2853676792, 0.2562656162,
          0.2301351374, 0.2067046525, 0.1857156162, 0.1669259363, 0.1501117216,
          0.1350678661, 0.1216078408, 0.1095629715, 0.0987814039, 0.08912689602,
          0.08047753984, 0.07272447676, 0.06577065171, 0.05952963239, 0.05392450919,
          0.04888688294, 0.04435594219, 0.04027762813, 0.03660388327, 0.03329197815,
          0.03030391032, 0.02760586906, 0.02516775982, 0.02296278221, 0.02096705614,
          0.01915929081, 0.01752049186, 0.0160337023, 0.01468377355, 0.0134571629,
          0.0123417545, 0.01132670097, 0.01040228333, 0.009559787048, 0.008791392283,
          0.008090076685, 0.007449529245, 0.006864073917, 0.006328601845, 0.005838511204,
          0.005389653752, 0.004978287316, 0.004601033533, 0.004254840222, 0.003936947878,
          0.003644859791],
        [
          0.0, 0.01170034459, 0.02335012948, 0.03449888978, 0.04420937286,
          0.05165884383, 0.05651665921, 0.05890350242, 0.05920200961, 0.057887842,
          0.05542416964, 0.05221048285, 0.04856616611, 0.04473272198, 0.04088425561,
          0.03714037829, 0.03357862388, 0.03024516387, 0.02716349942, 0.02434123292,
          0.02177519096, 0.01945521033, 0.01736687854, 0.01549347748, 0.01381733169,
          0.01232071857, 0.01098646129, 0.009798294828, 0.008741072509, 0.007800862276,
          0.0069649685, 0.00622190502, 0.005561337691, 0.004974009224, 0.004451655191,
          0.003986917157, 0.003573256901, 0.003204874224, 0.002876629804, 0.002583973901,
          0.002322881187, 0.002089791707, 0.001881557734, 0.001695396187, 0.001528846211,
          0.00137973146, 0.001246126682, 0.001126328134, 0.00101882747, 0.0009222886913,
          0.0008355278314, 0.000757495066, 0.0006872589573, 0.0006239925867, 0.0005669613501,
          0.0005155122161, 0.0004690642709, 0.0004271003938, 0.0003891599268, 0.0003548322166,
          0.0003237509226],
        0.04763834927, 0.0325232335))
```

---

## Subproblem 2 — Solution

```python
import math
import numbers
import numpy as np
from scipy.integrate import quad
from scipy.optimize import brentq


def _sphere_profile(y):
    # F(y) = 2 int_0^1 (1 - 3t/2 + t^3/2) cos(y t) dt: line profile of one sphere
    # of diameter D in units of D, with y = 2 pi D s (F(0) = 3/4). The closed
    # form 3/y^2 - 6 sin(y)/y^3 + 6 (1 - cos y)/y^4 cancels catastrophically for
    # small y, where the Taylor series is used instead.
    y = abs(y)
    if y < 0.5:
        total, term = 0.0, 1.0                       # term = (-1)^k y^(2k) / (2k)!
        for k in range(12):
            total += term * (1.0 / (2 * k + 1) - 1.5 / (2 * k + 2) + 0.5 / (2 * k + 4))
            term *= -y * y / ((2 * k + 1) * (2 * k + 2))
        return 2.0 * total
    return 3.0 / y ** 2 - 6.0 * math.sin(y) / y ** 3 + 6.0 * (1.0 - math.cos(y)) / y ** 4


def _profile(s, D_median, sigma):
    # I(s) = int A_model(|L|) cos(2 pi L s) dL. A_model is the volume-weighted
    # average of the single-sphere coefficients, so I(s) is the same average of
    # the single-sphere profiles D F(2 pi D s). The volume-weighted diameters are
    # log-normal with median D_median exp(3 sigma^2) and the same sigma.
    if sigma == 0.0:
        return D_median * _sphere_profile(2.0 * math.pi * D_median * s)
    D_vol = D_median * math.exp(3.0 * sigma ** 2)

    def integrand(z):
        D = D_vol * math.exp(sigma * z)
        return math.exp(-0.5 * z * z) * D * _sphere_profile(2.0 * math.pi * D * s)

    val = quad(integrand, -12.0, 12.0, epsabs=0.0, epsrel=1e-13, limit=400)[0]
    return val / math.sqrt(2.0 * math.pi)


def _validate(L_values, A_size, fit_index_lo, fit_index_hi):
    L = np.asarray(L_values, dtype=float)
    A = np.asarray(A_size, dtype=float)
    if L.ndim != 1 or A.ndim != 1 or L.size != A.size or L.size < 2:
        raise ValueError("L_values and A_size must be 1-D arrays of equal length >= 2")
    if not (np.all(np.isfinite(L)) and np.all(np.isfinite(A))):
        raise ValueError("L_values and A_size must be finite")
    if L[0] != 0.0 or np.any(np.diff(L) <= 0.0):
        raise ValueError("L_values must start at 0 and increase strictly")
    if A[0] != 1.0:
        raise ValueError("A_size[0] must be 1")
    for idx in (fit_index_lo, fit_index_hi):
        if isinstance(idx, (bool, np.bool_)) or not isinstance(idx, numbers.Integral):
            raise ValueError("fit indices must be integers")
    if not (0 <= fit_index_lo < fit_index_hi < L.size):
        raise ValueError("fit indices must satisfy 0 <= lo < hi < len(L_values)")
    return L, A


def subproblem_2(L_values, A_size, fit_index_lo, fit_index_hi):
    """
    Mean column lengths, log-normal sphere-size distribution and predicted
    line-profile widths from A_S(L).

    Inputs:
        L_values: 1-D array of column lengths L in nm, increasing, with
            L_values[0] = 0.
        A_size: 1-D array of size Fourier coefficients A_S(L) at L_values
            (A_size[0] = 1); entries may be 0 where the coefficient was lost
            in the noise.
        fit_index_lo: int, zero-based index of the first point of the linear
            fit for the area-weighted mean column length.
        fit_index_hi: int, zero-based index of the last point of that fit
            (inclusive).

    Output:
        Tuple (L_area, L_vol, D_median, sigma, A_model, p_col, beta, fwhm):
            L_area: float, area-weighted mean column length in nm.
            L_vol: float, volume-weighted mean column length in nm.
            D_median: float, median of the log-normal number distribution of
                sphere diameters in nm.
            sigma: float, logarithmic standard deviation of that distribution
                (dimensionless, >= 0).
            A_model: 1-D numpy array, size Fourier coefficient of a powder of
                spheres with that diameter distribution, at L_values.
            p_col: 1-D numpy array, column-length distribution p(L) of that
                powder in nm^-1 (unit integral), at L_values.
            beta: float, integral breadth of the size-broadened line profile
                of that powder, in nm^-1.
            fwhm: float, full width at half maximum of that profile, in nm^-1.

    Raises:
        ValueError under the conditions stated in the prompt.
    """
    L, A = _validate(L_values, A_size, fit_index_lo, fit_index_hi)

    # Area-weighted mean column length: unweighted straight line through the
    # hook-free points, extrapolated to A_S = 0
    L_fit = L[fit_index_lo:fit_index_hi + 1]
    A_fit = A[fit_index_lo:fit_index_hi + 1]
    L_mean, A_mean = np.mean(L_fit), np.mean(A_fit)
    c1 = np.sum((L_fit - L_mean) * (A_fit - A_mean)) / np.sum((L_fit - L_mean) ** 2)
    c0 = A_mean - c1 * L_mean
    if not (c1 < 0.0 and c0 > 0.0):
        raise ValueError("the fitted line does not reach A_S = 0 at a positive L")
    L_area = -c0 / c1

    # Volume-weighted mean column length: twice the trapezoidal area under A_S
    L_vol = 2.0 * np.sum(0.5 * (A[1:] + A[:-1]) * np.diff(L))

    # Spheres: <L>_area = (2/3) <D^3>/<D^2>, <L>_vol = (3/4) <D^4>/<D^3>.
    # Log-normal moments <D^k> = D_med^k exp(k^2 sigma^2 / 2) give
    # <L>_area = (2/3) D_med exp(5 sigma^2/2), <L>_vol = (3/4) D_med exp(7 sigma^2/2),
    # so <L>_vol/<L>_area = (9/8) exp(sigma^2); ratio <= 9/8 -> sigma = 0.
    sigma2 = max(math.log(8.0 * L_vol / (9.0 * L_area)), 0.0)
    sigma = math.sqrt(sigma2)
    D_median = 1.5 * L_area * math.exp(-2.5 * sigma2)

    # Size coefficient of the sphere powder. One sphere of diameter D:
    # A(L) = 1 - 3L/(2D) + L^3/(2D^3) for L < D, else 0. The powder average is
    # volume-weighted: A_S(L) = [M3(L) - (3L/2) M2(L) + (L^3/2) M0(L)] / M3(0)
    # with truncated moments M_n(L) = int_L^inf D^n f(D) dD, which for the
    # log-normal are D_med^n exp(n^2 sigma^2/2) Q_n,
    # Q_n = 0.5 erfc([ln(L/D_med) - n sigma^2] / (sigma sqrt 2)).
    #
    # Column-length distribution: a sphere of diameter D has p_D(l) = 2l/D^2 for
    # l < D (columns counted by projected area), and the powder average weights
    # each sphere by its number of columns (~ D^2):
    # p(l) = 2 l M0(l) / <D^2> = 2 l Q_0 / (D_med^2 exp(2 sigma^2)); p = <l> A_S''.
    A_model = np.empty_like(L)
    p_col = np.empty_like(L)
    for i, L_val in enumerate(L):
        if L_val == 0.0:
            A_model[i] = 1.0
            p_col[i] = 0.0
            continue
        x = L_val / D_median
        if sigma == 0.0:                       # monodisperse spheres
            A_model[i] = 1.0 - 1.5 * x + 0.5 * x ** 3 if x < 1.0 else 0.0
            p_col[i] = 2.0 * x / D_median if x < 1.0 else 0.0
            continue
        u = math.log(x)
        q = [0.5 * math.erfc((u - n * sigma2) / (sigma * math.sqrt(2.0))) for n in (0, 2, 3)]
        A_model[i] = (q[2]
                      - 1.5 * x * math.exp(-2.5 * sigma2) * q[1]
                      + 0.5 * x ** 3 * math.exp(-4.5 * sigma2) * q[0])
        p_col[i] = 2.0 * x * q[0] / (D_median * math.exp(2.0 * sigma2))

    # Predicted line profile I(s) = int A_model(|L|) cos(2 pi L s) dL. Its area is
    # A_model(0) = 1 and its peak is I(0) = 2 int_0^inf A_model dL, the
    # volume-weighted mean column length OF THE MODEL, (3/4) D_med exp(7 sigma^2/2).
    # This equals the measured L_vol only when sigma > 0; for sigma = 0 it is
    # (3/4) D_med = (9/8) L_area.
    I0 = 0.75 * D_median * math.exp(3.5 * sigma2)
    beta = 1.0 / I0

    # Half width: smallest s > 0 with I(s) = I(0)/2. I decreases from s = 0;
    # step outwards from below the root until the sign changes, then solve.
    D_vol = D_median * math.exp(3.0 * sigma2)
    half = lambda s: _profile(s, D_median, sigma) - 0.5 * I0
    step = 0.05 / D_vol
    s_lo, s_hi = 0.0, step
    while half(s_hi) > 0.0:
        s_lo, s_hi = s_hi, s_hi + step
    s_half = brentq(half, s_lo, s_hi, xtol=1e-15, rtol=1e-14, maxiter=500)
    fwhm = 2.0 * s_half

    return (float(L_area), float(L_vol), float(D_median), float(sigma), A_model, p_col,
            float(beta), float(fwhm))
```

---

## Main Problem — Solution

```python
import math
import numbers
import numpy as np
from scipy.integrate import quad
from scipy.optimize import brentq

def subproblem_1(two_theta_meas_deg, intensity_meas, two_theta_inst_deg,
                 intensity_inst, two_theta0_deg, wavelength_ka1_nm,
                 wavelength_ka2_nm, ka2_ratio, L_values):
    """
    Instrument- and doublet-corrected Fourier cosine coefficients of one line.

    Inputs:
        two_theta_meas_deg: 1-D array of 2theta positions in degrees
            (increasing) of the specimen step scan, recorded with
            K-alpha1 + K-alpha2 radiation.
        intensity_meas: 1-D array of specimen counts (intensity per unit
            2theta, including a linear background), same length as
            two_theta_meas_deg.
        two_theta_inst_deg: 1-D array of 2theta positions in degrees
            (increasing) of the instrumental-profile scan, which contains
            K-alpha1 only; its range and step may differ from those of the
            specimen scan.
        intensity_inst: 1-D array of instrumental counts, same length as
            two_theta_inst_deg.
        two_theta0_deg: float, reference angle 2theta_0 in degrees (K-alpha1
            peak position) that defines s = 0 for both scans.
        wavelength_ka1_nm: float, K-alpha1 wavelength in nm.
        wavelength_ka2_nm: float, K-alpha2 wavelength in nm.
        ka2_ratio: float, intensity ratio R = I(K-alpha2)/I(K-alpha1),
            0 <= R < 1.
        L_values: 1-D array of column lengths L in nm (L >= 0) at which the
            coefficients are required; L = 0 need not be included. At every
            requested L the transform of the instrumental profile is non-zero.

    Output:
        Tuple (A, sigma_A) of 1-D numpy arrays, one value per entry of
        L_values and in the same order:
            A: normalised Fourier cosine coefficients of the K-alpha1 physical
                line profile (A = 1 at L = 0).
            sigma_A: standard uncertainty of A from counting statistics
                (Poisson variance of every recorded count equal to the count,
                all counts independent), propagated to first order through
                the whole calculation (sigma_A = 0 at L = 0).
    """
    L = np.atleast_1d(np.asarray(L_values, dtype=float))
    lam1 = float(wavelength_ka1_nm)
    lam2 = float(wavelength_ka2_nm)
    R = float(ka2_ratio)
    sin_theta0 = np.sin(np.deg2rad(float(two_theta0_deg)) / 2.0)
    omega = 2.0 * np.pi * L

    def transform_weights(s):
        # Complex weights W with T(L) = W @ y, the exact integral of
        # exp(-i omega s) times the piecewise-linear interpolant of y(s).
        # On a segment of width h starting at s_a the integral is
        # h exp(-i omega s_a) [y_a (g0 - g1) + y_b g1], theta = omega h,
        # g0 = int_0^1 exp(-i theta u) du, g1 = int_0^1 u exp(-i theta u) du.
        h = np.diff(s)
        theta = np.outer(omega, h)
        g0 = np.empty(theta.shape, dtype=complex)
        g1 = np.empty(theta.shape, dtype=complex)
        small = np.abs(theta) < 0.05
        t = theta[~small]
        e = np.exp(-1j * t)
        g0[~small] = (1.0 - e) / (1j * t)
        g1[~small] = (e * (1.0 + 1j * t) - 1.0) / t ** 2
        # Taylor series where the closed forms lose precision (and at theta = 0)
        z = -1j * theta[small]
        s0 = np.ones_like(z)
        s1 = np.full_like(z, 0.5)
        fact = 1.0
        for k in range(1, 10):
            fact *= k
            zk = z ** k
            s0 += zk / (fact * (k + 1))
            s1 += zk / (fact * (k + 2))
        g0[small] = s0
        g1[small] = s1
        seg = np.exp(-1j * np.outer(omega, s[:-1])) * h
        W = np.zeros((omega.size, s.size), dtype=complex)
        W[:, :-1] += seg * (g0 - g1)
        W[:, 1:] += seg * g1
        return W

    def normalised_transform(two_theta_deg, counts):
        # Returns the normalised transform and its derivatives with respect
        # to every recorded count (the calculation is linear in the counts
        # up to the final normalisation).
        tt = np.asarray(two_theta_deg, dtype=float)
        n = np.asarray(counts, dtype=float)
        u = (tt - tt[0]) / (tt[-1] - tt[0])
        # Straight-line background through the first and last points (in 2theta)
        I_net = n - ((1.0 - u) * n[0] + u * n[-1])
        theta = np.deg2rad(tt) / 2.0
        s = 2.0 * (np.sin(theta) - sin_theta0) / lam1
        # Intensity per unit s: I_s = I_2theta |d(2theta)/ds| = I_2theta lam1 / cos(theta)
        c = lam1 / np.cos(theta)
        W = transform_weights(s) * c               # T(L) = W @ I_net
        h = np.diff(s)
        w0 = np.zeros(s.size)
        w0[:-1] += 0.5 * h
        w0[1:] += 0.5 * h
        w0 = w0 * c                                # T(0) = w0 @ I_net (exact)
        # Linear functionals of the raw counts: T = a @ n and T0 = b @ n,
        # with the end-point counts entering through the background line
        a = W.copy()
        a[:, 0] -= W @ (1.0 - u)
        a[:, -1] -= W @ u
        b = w0.copy()
        b[0] -= w0 @ (1.0 - u)
        b[-1] -= w0 @ u
        T = W @ I_net
        T0 = w0 @ I_net
        F = T / T0
        dF = (a - np.outer(F, b)) / T0            # dF/dn_j
        return F, dF, n

    H, dH, n_h = normalised_transform(two_theta_meas_deg, intensity_meas)
    G, dG, n_g = normalised_transform(two_theta_inst_deg, intensity_inst)

    # K-alpha2 replica: displaced in s to the K-alpha2 Bragg position,
    # Delta = 2 sin(theta0) (lam2/lam1 - 1) / lam1, weight R; its normalised
    # transform (1 + R exp(-2 pi i L Delta)) / (1 + R) multiplies the specimen
    # transform, since only the specimen scan contains the doublet.
    delta = 2.0 * sin_theta0 * (lam2 / lam1 - 1.0) / lam1
    D = (1.0 + R * np.exp(-1j * omega * delta)) / (1.0 + R)

    # Stokes deconvolution with the complete complex transforms
    Fphys = H / (G * D)
    A = np.real(Fphys)

    # First-order propagation of the Poisson variances (var n_j = n_j) of
    # the two independent scans; A = Re F, so only the real parts of the
    # complex derivatives enter.
    dA_h = np.real(dH / (G * D)[:, None])
    dA_g = np.real(-(Fphys / G)[:, None] * dG)
    sigma_A = np.sqrt(dA_h ** 2 @ n_h + dA_g ** 2 @ n_g)
    sigma_A[L == 0.0] = 0.0                      # A(0) = 1 identically
    return A, sigma_A


def _sphere_profile(y):
    # F(y) = 2 int_0^1 (1 - 3t/2 + t^3/2) cos(y t) dt: line profile of one sphere
    # of diameter D in units of D, with y = 2 pi D s (F(0) = 3/4). The closed
    # form 3/y^2 - 6 sin(y)/y^3 + 6 (1 - cos y)/y^4 cancels catastrophically for
    # small y, where the Taylor series is used instead.
    y = abs(y)
    if y < 0.5:
        total, term = 0.0, 1.0                       # term = (-1)^k y^(2k) / (2k)!
        for k in range(12):
            total += term * (1.0 / (2 * k + 1) - 1.5 / (2 * k + 2) + 0.5 / (2 * k + 4))
            term *= -y * y / ((2 * k + 1) * (2 * k + 2))
        return 2.0 * total
    return 3.0 / y ** 2 - 6.0 * math.sin(y) / y ** 3 + 6.0 * (1.0 - math.cos(y)) / y ** 4


def _profile(s, D_median, sigma):
    # I(s) = int A_model(|L|) cos(2 pi L s) dL. A_model is the volume-weighted
    # average of the single-sphere coefficients, so I(s) is the same average of
    # the single-sphere profiles D F(2 pi D s). The volume-weighted diameters are
    # log-normal with median D_median exp(3 sigma^2) and the same sigma.
    if sigma == 0.0:
        return D_median * _sphere_profile(2.0 * math.pi * D_median * s)
    D_vol = D_median * math.exp(3.0 * sigma ** 2)

    def integrand(z):
        D = D_vol * math.exp(sigma * z)
        return math.exp(-0.5 * z * z) * D * _sphere_profile(2.0 * math.pi * D * s)

    val = quad(integrand, -12.0, 12.0, epsabs=0.0, epsrel=1e-13, limit=400)[0]
    return val / math.sqrt(2.0 * math.pi)


def _validate(L_values, A_size, fit_index_lo, fit_index_hi):
    L = np.asarray(L_values, dtype=float)
    A = np.asarray(A_size, dtype=float)
    if L.ndim != 1 or A.ndim != 1 or L.size != A.size or L.size < 2:
        raise ValueError("L_values and A_size must be 1-D arrays of equal length >= 2")
    if not (np.all(np.isfinite(L)) and np.all(np.isfinite(A))):
        raise ValueError("L_values and A_size must be finite")
    if L[0] != 0.0 or np.any(np.diff(L) <= 0.0):
        raise ValueError("L_values must start at 0 and increase strictly")
    if A[0] != 1.0:
        raise ValueError("A_size[0] must be 1")
    for idx in (fit_index_lo, fit_index_hi):
        if isinstance(idx, (bool, np.bool_)) or not isinstance(idx, numbers.Integral):
            raise ValueError("fit indices must be integers")
    if not (0 <= fit_index_lo < fit_index_hi < L.size):
        raise ValueError("fit indices must satisfy 0 <= lo < hi < len(L_values)")
    return L, A


def subproblem_2(L_values, A_size, fit_index_lo, fit_index_hi):
    """
    Mean column lengths, log-normal sphere-size distribution and predicted
    line-profile widths from A_S(L).

    Inputs:
        L_values: 1-D array of column lengths L in nm, increasing, with
            L_values[0] = 0.
        A_size: 1-D array of size Fourier coefficients A_S(L) at L_values
            (A_size[0] = 1); entries may be 0 where the coefficient was lost
            in the noise.
        fit_index_lo: int, zero-based index of the first point of the linear
            fit for the area-weighted mean column length.
        fit_index_hi: int, zero-based index of the last point of that fit
            (inclusive).

    Output:
        Tuple (L_area, L_vol, D_median, sigma, A_model, p_col, beta, fwhm):
            L_area: float, area-weighted mean column length in nm.
            L_vol: float, volume-weighted mean column length in nm.
            D_median: float, median of the log-normal number distribution of
                sphere diameters in nm.
            sigma: float, logarithmic standard deviation of that distribution
                (dimensionless, >= 0).
            A_model: 1-D numpy array, size Fourier coefficient of a powder of
                spheres with that diameter distribution, at L_values.
            p_col: 1-D numpy array, column-length distribution p(L) of that
                powder in nm^-1 (unit integral), at L_values.
            beta: float, integral breadth of the size-broadened line profile
                of that powder, in nm^-1.
            fwhm: float, full width at half maximum of that profile, in nm^-1.

    Raises:
        ValueError under the conditions stated in the prompt.
    """
    L, A = _validate(L_values, A_size, fit_index_lo, fit_index_hi)

    # Area-weighted mean column length: unweighted straight line through the
    # hook-free points, extrapolated to A_S = 0
    L_fit = L[fit_index_lo:fit_index_hi + 1]
    A_fit = A[fit_index_lo:fit_index_hi + 1]
    L_mean, A_mean = np.mean(L_fit), np.mean(A_fit)
    c1 = np.sum((L_fit - L_mean) * (A_fit - A_mean)) / np.sum((L_fit - L_mean) ** 2)
    c0 = A_mean - c1 * L_mean
    if not (c1 < 0.0 and c0 > 0.0):
        raise ValueError("the fitted line does not reach A_S = 0 at a positive L")
    L_area = -c0 / c1

    # Volume-weighted mean column length: twice the trapezoidal area under A_S
    L_vol = 2.0 * np.sum(0.5 * (A[1:] + A[:-1]) * np.diff(L))

    # Spheres: <L>_area = (2/3) <D^3>/<D^2>, <L>_vol = (3/4) <D^4>/<D^3>.
    # Log-normal moments <D^k> = D_med^k exp(k^2 sigma^2 / 2) give
    # <L>_area = (2/3) D_med exp(5 sigma^2/2), <L>_vol = (3/4) D_med exp(7 sigma^2/2),
    # so <L>_vol/<L>_area = (9/8) exp(sigma^2); ratio <= 9/8 -> sigma = 0.
    sigma2 = max(math.log(8.0 * L_vol / (9.0 * L_area)), 0.0)
    sigma = math.sqrt(sigma2)
    D_median = 1.5 * L_area * math.exp(-2.5 * sigma2)

    # Size coefficient of the sphere powder. One sphere of diameter D:
    # A(L) = 1 - 3L/(2D) + L^3/(2D^3) for L < D, else 0. The powder average is
    # volume-weighted: A_S(L) = [M3(L) - (3L/2) M2(L) + (L^3/2) M0(L)] / M3(0)
    # with truncated moments M_n(L) = int_L^inf D^n f(D) dD, which for the
    # log-normal are D_med^n exp(n^2 sigma^2/2) Q_n,
    # Q_n = 0.5 erfc([ln(L/D_med) - n sigma^2] / (sigma sqrt 2)).
    #
    # Column-length distribution: a sphere of diameter D has p_D(l) = 2l/D^2 for
    # l < D (columns counted by projected area), and the powder average weights
    # each sphere by its number of columns (~ D^2):
    # p(l) = 2 l M0(l) / <D^2> = 2 l Q_0 / (D_med^2 exp(2 sigma^2)); p = <l> A_S''.
    A_model = np.empty_like(L)
    p_col = np.empty_like(L)
    for i, L_val in enumerate(L):
        if L_val == 0.0:
            A_model[i] = 1.0
            p_col[i] = 0.0
            continue
        x = L_val / D_median
        if sigma == 0.0:                       # monodisperse spheres
            A_model[i] = 1.0 - 1.5 * x + 0.5 * x ** 3 if x < 1.0 else 0.0
            p_col[i] = 2.0 * x / D_median if x < 1.0 else 0.0
            continue
        u = math.log(x)
        q = [0.5 * math.erfc((u - n * sigma2) / (sigma * math.sqrt(2.0))) for n in (0, 2, 3)]
        A_model[i] = (q[2]
                      - 1.5 * x * math.exp(-2.5 * sigma2) * q[1]
                      + 0.5 * x ** 3 * math.exp(-4.5 * sigma2) * q[0])
        p_col[i] = 2.0 * x * q[0] / (D_median * math.exp(2.0 * sigma2))

    # Predicted line profile I(s) = int A_model(|L|) cos(2 pi L s) dL. Its area is
    # A_model(0) = 1 and its peak is I(0) = 2 int_0^inf A_model dL, the
    # volume-weighted mean column length OF THE MODEL, (3/4) D_med exp(7 sigma^2/2).
    # This equals the measured L_vol only when sigma > 0; for sigma = 0 it is
    # (3/4) D_med = (9/8) L_area.
    I0 = 0.75 * D_median * math.exp(3.5 * sigma2)
    beta = 1.0 / I0

    # Half width: smallest s > 0 with I(s) = I(0)/2. I decreases from s = 0;
    # step outwards from below the root until the sign changes, then solve.
    D_vol = D_median * math.exp(3.0 * sigma2)
    half = lambda s: _profile(s, D_median, sigma) - 0.5 * I0
    step = 0.05 / D_vol
    s_lo, s_hi = 0.0, step
    while half(s_hi) > 0.0:
        s_lo, s_hi = s_hi, s_hi + step
    s_half = brentq(half, s_lo, s_hi, xtol=1e-15, rtol=1e-14, maxiter=500)
    fwhm = 2.0 * s_half

    return (float(L_area), float(L_vol), float(D_median), float(sigma), A_model, p_col,
            float(beta), float(fwhm))


def main_problem(two_theta_meas_deg, intensity_meas, two_theta_inst_deg,
                 intensity_inst, two_theta0_deg, wavelength_ka1_nm,
                 wavelength_ka2_nm, ka2_ratio, m_values, d1_nm, L_values,
                 fit_index_lo, fit_index_hi):
    """
    Warren-Averbach line-profile analysis of a nanocrystalline powder.

    Inputs:
        two_theta_meas_deg: sequence of n 1-D arrays; element j holds the
            2theta positions in degrees (increasing) of the specimen scan of
            order m_values[j] (K-alpha1 + K-alpha2 radiation).
        intensity_meas: sequence of n 1-D arrays of specimen counts (per unit
            2theta, including background and noise), matching
            two_theta_meas_deg.
        two_theta_inst_deg: sequence of n 1-D arrays of 2theta positions in
            degrees (increasing) of the K-alpha1-only instrumental scans.
        intensity_inst: sequence of n 1-D arrays of instrumental counts,
            matching two_theta_inst_deg.
        two_theta0_deg: sequence of n floats; element j is the reference angle
            2theta_0 in degrees (K-alpha1 peak position) shared by the two
            scans of order m_values[j].
        wavelength_ka1_nm: float, K-alpha1 wavelength in nm.
        wavelength_ka2_nm: float, K-alpha2 wavelength in nm.
        ka2_ratio: float, intensity ratio R = I(K-alpha2)/I(K-alpha1),
            0 <= R < 1.
        m_values: sequence of n distinct positive integer orders.
        d1_nm: float, interplanar spacing of the first-order reflection in nm.
        L_values: 1-D array of column lengths in nm, increasing, with
            L_values[0] = 0; at every L the transforms of the instrumental
            profiles are non-zero.
        fit_index_lo: int, zero-based index of the first L_values point in
            the linear fit for the area-weighted column length.
        fit_index_hi: int, zero-based index of the last point in that fit
            (inclusive).

    Output:
        Tuple (L_area, L_vol, D_median, sigma, eps_rms, A_model):
            L_area: float, area-weighted mean column length in nm.
            L_vol: float, volume-weighted mean column length in nm.
            D_median: float, median of the log-normal number distribution of
                sphere diameters in nm.
            sigma: float, logarithmic standard deviation of that distribution
                (dimensionless, >= 0).
            eps_rms: 1-D numpy array of root-mean-square strains
                (dimensionless), one per entry of L_values.
            A_model: 1-D numpy array, size Fourier coefficient of the
                log-normal sphere powder, one per entry of L_values.
    """
    L = np.asarray(L_values, dtype=float)
    m_squared = np.asarray(m_values, dtype=float) ** 2

    # Step 1: corrected Fourier coefficients of every order on the common L grid
    A_orders = np.array([
        subproblem_1(two_theta_meas_deg[j], intensity_meas[j],
                     two_theta_inst_deg[j], intensity_inst[j],
                     two_theta0_deg[j], wavelength_ka1_nm, wavelength_ka2_nm,
                     ka2_ratio, L)[0]
        for j in range(len(m_values))
    ])

    # Step 2: weighted Warren-Averbach separation, ln A = ln A_S - 2 pi^2 m^2 L^2 <eps^2> / d1^2,
    # weights A^2 (inverse variance of ln A for equal absolute uncertainty of A)
    A_size = np.zeros_like(L)
    eps_rms = np.zeros_like(L)
    for i, L_val in enumerate(L):
        if L_val == 0.0:
            A_size[i] = 1.0
            continue
        coeffs = A_orders[:, i]
        usable = coeffs > 0.0
        if np.count_nonzero(usable) < 2:
            continue                                   # A_S = 0, eps_rms = 0
        x = m_squared[usable]
        y = np.log(coeffs[usable])
        w = coeffs[usable] ** 2
        W = np.sum(w)
        x_bar = np.sum(w * x) / W
        y_bar = np.sum(w * y) / W
        b = np.sum(w * (x - x_bar) * (y - y_bar)) / np.sum(w * (x - x_bar) ** 2)
        a = y_bar - b * x_bar
        A_size[i] = np.exp(a)
        eps2 = -b * d1_nm ** 2 / (2.0 * np.pi ** 2 * L_val ** 2)
        eps_rms[i] = np.sqrt(eps2) if eps2 > 0.0 else 0.0

    # Step 3: mean column lengths, log-normal spheres and their size coefficient
    L_area, L_vol, D_median, sigma, A_model = subproblem_2(L, A_size, fit_index_lo, fit_index_hi)[:5]

    return L_area, L_vol, D_median, sigma, eps_rms, A_model
```
