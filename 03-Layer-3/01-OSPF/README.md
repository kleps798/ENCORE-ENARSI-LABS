This ia a lab where i learnt about OSPF Area types and labed them  The area types demonstrated are:

1. Stub Area
2. Totally Stub Area
3. NSSA
4. Totally NSSA.

It was interesting to learn how they work and i finally managed to do the disctions below:

OSPF area types 

Area type	Inter-area routes	External routes	                                     Local redistribution
Normal	           Yes	              Type 5 allowed	                                              Yes
Stub	           Yes	              Type 5 blocked	                                              No
Totally stubby	   Default route,     individual inter-area routes suppresseType 5 blocked            No
NSSA	           Yes	             Type 5 blocked from entering; local externals use Type 7	      Yes
Totally NSSA	Default route,        individual inter-area routes suppressedType 5 blocked from entering; local externals use Type 7	Yes

 
