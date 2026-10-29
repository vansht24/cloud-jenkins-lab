# Validation performed

Tested on Windows with Temurin JDK 21 and Jenkins LTS 2.580.1.

- Jenkins build 1: SUCCESS; 8 passed, 0 failed.
- Jenkins build 2: FAILURE; 6 passed, 2 failed.
- Jenkins build 3: SUCCESS; 8 passed, 0 failed.

The bug and fix builds were triggered by real SCM polling after local commits, not by manually pressing Build Now. Successful builds archived a runnable JAR; the failed build published no new artifact. The built-in controller node had zero executors; a separate local agent performed the builds.

PowerShell launchers were syntax checked; the Git server was exercised. Bash launchers were syntax checked with Git Bash. macOS and Linux end-to-end execution was not performed; those instructions/scripts use the same Java project and standard Bash, curl, and Git interfaces.

This is a trusted, loopback-only teaching lab, not a production security baseline.
