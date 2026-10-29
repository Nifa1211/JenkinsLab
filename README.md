# FestPass — Jenkins CI Assignment Evidence

## Overview

This repository demonstrates a local Jenkins CI pipeline for the FestPass Java application. Jenkins checks out the `main` branch from the local Git server, runs the supplied test suite, packages a JAR, smoke-tests the real JAR, and archives build artifacts.

The SCM polling trigger is configured in `Jenkinsfile`:

```groovy
triggers {
    pollSCM('* * * * *')
}
```

This lab uses a local Git server (`git://127.0.0.1:9418/festpass-java`); a GitHub push is not required for Jenkins polling in the lab.

## Build evidence

| Build | Result | Commit | Evidence |
|---|---|---|---|
| #9 | FAILURE — expected red build | `24fceb3beddc32f1824c0993db4ff37b43f414fa` | [Automatic red build screenshot](screenshots/build-09-automatic-red.png) |
| #10 | SUCCESS — green build | `634f6d4cef3a0b0b6cd742b89e7a932e8a42558a` | [Jenkins job overview](screenshots/jenkins-job-overview-build-10.png) |

Build #9's screenshot shows **Started by an SCM change** and the three cancellation-related failures:
- `FestPassTest.cancellationFreesSeat`
- `FestPassTest.cancellationsSurviveReload`
- `FestPassTest.cliWorkflowPersistsAcrossCommands`

Build #10's evidence CSV records **28 tests, 0 failures, 0 errors, 0 skipped**, with result `SUCCESS`. The pipeline produced `build/festpass.jar` and completed seven real-JAR smoke-test commands.

> For the final assessment, open Build #10 in Jenkins and confirm its build cause says **Started by an SCM change**. The supplied job-overview screenshot shows the green Build #10 but does not display its cause.

## Diagnosis and fix

The defect was in `BookingService.cancel()`: it checked whether the student had a booking but did not remove the student from the event's roster. The fix removes the student and preserves the error for an unknown booking:

```java
if (!event.students.remove(studentId)) {
    throw new IllegalArgumentException(
        "No booking for student: " + studentId
    );
}
```

This restored the cancellation behavior, including freeing a seat, persisting cancellation across reloads, and reflecting the cancellation in the CLI workflow. The supplied tests were not changed.

## Green-build artifacts and JAR verification

- `festpass.jar` — downloaded from Jenkins Build #10.
- `build-info.txt` — [screenshot](screenshots/build-10-build-info.png).
- `smoke-test.txt` — [screenshot](screenshots/build-10-smoke-test.png).
- `submission-evidence.csv` — downloaded from Build #10 separately.
- JAR help output — [screenshot](screenshots/final-jar-help-output.png).

The JAR was run locally with:

```powershell
java -jar .\festpass.jar help
```

It displayed the personalized introduction:

`FestPass - campus event registration | Maintained by Nifa1211`

## Submission checklist

- [x] SCM polling trigger committed in `Jenkinsfile`.
- [x] Automatic red build #9 observed.
- [x] Cancellation defect fixed in the application.
- [x] Green build #10 with 28 passing tests.
- [x] JAR packaged and seven real-JAR smoke-test commands passed.
- [ ] Save Build #9's `submission-evidence.csv` separately from Build #10's CSV.
- [ ] Verify Build #10's cause in its detail page and capture a screenshot if it says `Started by an SCM change`.
- [ ] Attach the final JAR and both evidence CSVs to the instructor's submission, as required.

## Git history

Relevant commits:

- `d70e6c7` — Configure Jenkins SCM polling
- `24fceb3` — Personalize help introduction (red-build commit)
- `634f6d4` — Fix booking cancellation (green-build commit)

## Public GitHub note

The Jenkins lab itself uses the local Git server and does not need GitHub. If publishing this project publicly, do **not** commit `submission.properties`, submission-evidence CSVs, or other instructor/private assessment details. Prefer a private GitHub repository for the assignment evidence, and check your course rules before publishing the supplied starter project.
