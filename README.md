# Amber_linmin
backup from amber website


# LINMIN failures
LINMIN failure is a common problem in minimization. For similar problems with dynamics/SHAKE, see the BLOWUP description. Note that nmode has a more robust minimizer.
User 1:
...the minimization calculation has always ended up with 'fatal error.' These are the messages in the output file:
 %MIN-W-PROBS, search restarted due to LINMIN failure
 %MIN-F-FAILURE, repeated line search failure
 %MINMD-F-ERSTOP, fatal error
It seems like when SHAKE is on, 'fatal error' occurs after fewer minimization steps.
User 2:
Some of the output files(the 'mdout' in the manual) contain the message like:
 .... RESTARTED DUE TO LINMIN FAILURE ...
the input is like:
 &cntrl                                                                        
    imin=1, maxcyc=10000, ncyc=100, idiel=0, cut=30.0, scee=1.2,               
 &end
But if I change the input into:
 &cntrl
    imin=1, maxcyc=10000, ncyc=1000, idiel=0, cut=30.0, scee=1.2,
 &end
there will be no "LINMIN FAILURE" message. The difference between two input file is only ncyc(from 100 to 1000). Both trials gave normal "FINAL RESULTS" Does it matter, if the mdout contains it? If so, how to correct it?
From: K Bryson 
Date: Fri, 18 Mar 1994 00:57:02 +0000 (GMT). 
I think I only remember one occasion when I had a problem minimising a periodic water box simulations when I was not using SHAKE although I cannot remember if it produced a LINMIN error. The reason turned out to be quite unusual.
Seemingly I had chosen wall dimensions such that I had two water molecules at opposite walls of the box which ( when you took the periodic BC into account ) had their OW atoms less than an angstrom apart ! The edit module although it considers solute-water steric clashing and deletes the water, it doesn't consider water-water steric clashing in the periodic case from opposite sides of the box. Since the water box is generated from smaller identical water boxes you get situations where periodic copies can clash with two identical cartesian coordinates and so the third coordinate just need to be close in a periodic sense.

What happened during minimisation was that the two oxygen atoms got shifted by enormous distances due to the steric clashing then the OW-HW bond lengths were enormous and then I remember some kind of minimisation error occurred soon afterwards. This would occur in the first few steps if SHAKE was applied since it would then try to iteratively constrain these enormous OW-HW lengths, unsuccessfully I would imagine !

I found that a work-around solution was simply to modify the X,Y and Z-cut distances in the edit module when you are constructing the water box by a few Angstroms. This usually got rid off the bad water sterics. [NOTE: it is easier and more reliable to manually add .4A or more to the box dimensions at the end of the prmtop file. In 4.1, EDIT automatically adds .4A.]

From: dap@portal.vpharm.com (David A. Pearlman)
Subject: LINMIN and SHAKE failures
Date: Fri, 18 Mar 1994 16:21:33 -0500 (EST)
The LINMIN failures occur frequently when performing minimization. They don't mean that any sort of evil failure of minimization has occurred, only that the minimizer got "stuck" in a place from which the minimization algorithm could not find a way out. Unless there is something very askew with your system, the amount of minimization that has occurred by the time you reach such a "sticking" point will be sufficient to move on to MD or Gibbs. (It wouldn't be sufficient to carry out a normal modes calculation, but a different minimizer is used then, anyway).
SHAKE has a propensity to fail when there are very large forces in the system. If your system is poorly defined at the onset, it is likely that it will contain a number of hot spots, where the initial forces will be very large. Attempting to use SHAKE at this point increases the chance that you will encounter a "fatal error" due to SHAKE. In general, it is not necesary to use SHAKE during minimization, even if you will subsequently be using SHAKE in MD or GIBBS. But if you want to use SHAKE, I would suggest that you first minimize the system without SHAKE (to remove any extreme hot spots), then run a second minimization with SHAKE on, starting with the coordinates from the first minimization. This 2-step process has a greater likelihood of working. Although, as noted before, I don't believe there is much value in minimizing with SHAKE on for most cases...
