We want to turn APT essentially into a collection of microservices of sorts that communicate with each other over clearly defined interfaces. In particular, we want to separate the logic from the presentation.

## Legacy frontend
The existing apt frontend will perform calculations, print them, and so on, but printing code is heavily intertwined with the logic of actually calculating them. If you look at say the autoremovals, they are being calculated right before they are printed. Crazy.

## Goal

### Separation of data/logic and presentation
We should redefine a command like install to first calculate an entire solution and then put that into a data structure. Following that, the frontend should render the data structure to the screen in the given [[output format]].

### Introducing a daemon
The new frontend should be talking to a daemon, [[aptd]]. Now, this cannot be always a separate process: For example, in a #chroot, there is no service manager. So we should be able to directly do the daemon things in process too.

### Separating privileged and uprivileged operations
The [[aptd]] daemon should have a companion daemon called [[apt-dpkgd]] that essentially receives and carries out the actual dpkg operations.