# EBTEL
Enthalpy-Based Thermal Evolution of Loops (EBTEL) model coded in the IDL language by J. A. Klimchuk, S. Patsourakos, and P.J. Cargill

For more information regarding the EBTEL model see:

+ <a href="http://adsabs.harvard.edu/abs/2008ApJ...682.1351K">Klimchuk et al. 2008, ApJ, 682:1351-1362 (Paper 1)</a>
+ <a href="http://adsabs.harvard.edu/abs/2012ApJ...752..161C">Cargill et al. 2012A, ApJ, 752:161 (Paper 2)</a>
+ <a href="http://adsabs.harvard.edu/abs/2012ApJ...758....5C">Cargill et al. 2012B, ApJ, 758:5 (Paper 3)</a>

This code is distributed as is. Modifications by the user are allowed, but are unsupported. If you believe you have encountered an error in the original (i.e. unaltered) code, create an issue or submit a pull request with the requested changes. 

##PURPOSE:
Compute the evolution of spatially-averaged (along the field) loop quantities using simplified equations.  The instantaneous differential emission measure of the transition region is also computed. This version incorporates all the modifications from Paper 2 and is written in modular form for clarity. DEM parts unchanged except for TR pressure correction (see Paper 2). 

##INPUTS:
+ ttime  = time array (s)
+ heat   = heating rate array (erg cm^-3 s^-1)   (direct heating only; the first element, heat(0), determines the initial static equilibrium)
+ length = loop half length (top of chromosphere to apex) (cm)

##OPTIONAL KEYWORD INPUTS:
+ classical = set to use the UNsaturated classical heat flux
+ dynamic   = set to use dynamical r1 and r2 (NOT recommended, especially when T > 10 MK). Now redundant.
+ dem_old   = set to use old technique of computing DEM(T) in the trans. reg. (weighted average of demev, demcon, and demeq)
+ flux_nt   = energy flux array for nonthermal electrons impinging the chromosphere (input as a positive quantity; erg cm^-2 s^-1)
+ energy_nt = mean energy of the nonthermal electrons in keV (default is 50 keV)
+ rtv       = set to use Rosner, Tucker, Vaiana radiative loss function (Winebarger form)

##OUTPUTS:
+ t (t_a) = temperature array corresponding to time (avg. over coronal section of loop / apex)
+ n (n_a) = electron number density array (cm^-3) (coronal avg. / apex)
+ p (p_a) = pressure array (dyn cm^-2) (coronal avg. /apex)
+ v = velocity array (cm s^-1) (r4 * velocity at base of corona)
+ c11 = C1 (or r3 in this code)
+ dem_tr = differential emission measure of transition region, dem(time,T), both legs (dem = n^2 * ds/dT  cm^-5 K^-1) (Note:  dem_tr is not reliable when a nonthermal electron flux is used.)
+ dem_cor = differential emission measure of corona, dem(time,T), both legs(Int{dem_cor+dem_tr dT} gives the total emission measure of a loop strand having a cross sectional area of 1 cm^2)
+ logtdem = logT array corresponding to dem_tr and dem_cor
+ f_ratio = ratio of heat flux to equilibrium heat flux (ratio of heat flux to tr. reg. radiative loss rate)
+ rad_ratio = ratio of tr. reg. radiative loss rate from dem_tr and from r3*(coronal rate)
+ cond = conductive loss from corona
+ rad_cor =  coronal radiative loss

##CORRESPONDENCE WITH VARIABLES IN ASTROPHYSICAL JOURNAL ARTICLES:
(i.e. Papers 1-3)

+ r1 = c_3
+ r2 = c_2
+ r3 = c_1
+ f, ff = F_0
+ f_eq, ff_eq = - R_tr
+ dem_eq = DEM_se

##USAGE:
+ To include the transition region DEM:
  + `IDL> ebtel2, ttime, heat, t, n, p, v, dem_tr, dem_cor, logtdem`
+ To exclude the transition region DEM (faster):
  + `IDL> ebtel2, ttime, heat, t, n, p, v`
+ To include a nonthermal electron energy flux:
  + `IDL> ebtel2, ttime, heat, t, n, p, v, flux_nt=flux_nt`
+ To compute rad_ratio (Requires 25% more computing time):
  + `IDL> ebtel2, ttime, heat, t, n, p, v, dem_tr, dem_cor, logtdem, f_ratio, rad_ratio`

##INTENSITIES:
For observations in which temperature response function, G(T), has units of DN s^-1 pix^-1 cm^5 and the loop diameter, d, is larger than the pixel dimension, l_pix:

+ I_cor_perp = d/(2L)* Int{G(T)*dem_cor(T)*dT}
+ I_tr_perp = d/l_pix * Int{G(T)*dem_tr(T)*dT}
+ I_tr_parallel = Int{G(T)*dem_tr(T)*dT} ,

for lines-of-sight perpendicular and parallel to the loop axis. I_tr_perp assumes that the transition region is thinner than l_pix.

##MISCELLANEOUS COMMENTS:
+ A 1 sec time is generally adequate, but in applications where exceptionally strong conductive cooling is expected (e.g., intense short duration heating events, especially in short loops), a shorter time step may be necessary. If there is a question, users should compare runs with different time steps and verify that there are no significant differences in the results.
+ Runs much more quickly if the transition region DEM is not computed.
+ Speed can be increased by increasing the minimum DEM temperature from 10^4 to, say, 10^5 Kor by decreasing the maximum DEM temperature from 10^8.5 to, say, 10^7.5 (search on 450 and 451).
+ The equilibrium base heat flux coefficient of 2/7 is appropriate for uniform heating; a coefficient of 4/7 is more appropriate for apex heating.
+ To have equal amounts of thermal and nonthermal heating:  flux_nt = heat*length.
+ It is desirable to have a low-level background heating during the cooling phase so that the coronal temperature does not drop below values at which the corona DEM is invalid.
+ r1 = c_3 = 0.7 gives more accurate coronal evolution than the original 0.5, especially in the late phase of cooling.  However, it produces excess DEM at the very hottest temperatures during impulsive events, and the transition region DEM is somewhat elevated.  We have therefore introduced r1_tr = 0.5, which provides a more accurate transition region DEM at the same time that r1 = 0.7 provides a more accurate radiative cooling.
+ v = (c_3/c_2)*(t_tr/t)*v_0 = (r1/r2)*(t_tr/t)*(v/r4) at temperature t_tr in the transition region, where t is the average coronal temperature.

 
##HISTORY:
+ May 2012. PC version. Modular form. 
+ 2013 Jan 15, JAK, Fixed a bug in the compution of the radiation function at 10^4 K; important for computing radiation losses based on dem_tr; ge vs. gt in computing rad;  lt vs. le in computing rad_dem