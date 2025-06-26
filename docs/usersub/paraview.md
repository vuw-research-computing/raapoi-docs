## ParaView (via OpenFOAM)

!!! Warning "Friendly Reminder"
    *HPC is built to serve powerful computational work which generally happens at pre-visualisation stage, and is not entirely meant to fulfill visualisation needs as discussed in [FAQ section: Visualisation](../faq.md#visualisation). Please proceed only if you think doing visualisation/plotting locally is not an option. Kindly note that these instructions are meant for Paraview usage alongside OpenFOAM.*

To connect ParaView to Rāpoi, you will need 3 terminal windows: two to extend the virtual handshake from Rāpoi and from your local computer, and one to open ParaView.

Terminal window 1:

1. Log in to Rāpoi.

2. Initiate an interactive job to avoid putting strain on the login node (assign desired cpus and memory).

3. Run command ``hostname -I | awk '{print $1}'`` to identify the IP address of the computing node.

4. Run command ``./pvserver``

Terminal window 2:

1. Run command ``ssh -N -L 11111:<IP address>:11111 <username>@raapoi.vuw.ac.nz`` (make sure to enter the IP address of the computing node you identified earlier).

Terminal window 3:

1. Source OpenFOAM from your local computer (make sure the OpenFOAM version you source is the same version as what is installed on Rāpoi)

2. Run command ``paraFoam`` to open ParaView.

3. On ParaView, select File -> Connect...

4. Highlight the desired server, and click Connect
