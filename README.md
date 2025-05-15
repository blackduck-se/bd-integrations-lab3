# Black Duck Software Integration Lab 3

The goal of this lab is to provide hands on experience integrating a **Coverity** scan into a **GitHub** workflow using the [Black Duck Security Scan Action](https://github.com/marketplace/actions/black-duck-security-scan) and demonstrating its post scan capabilities. As part of the lab, we will:
1. setup a GitHub workflow and execute a full scan
2. break the build based on a [view](https://documentation.blackduck.com/bundle/coverity-docs/page/coverity-platform/topics/view_management.html) defined in Coverity Connect
3. review the findings in Coverity Connect
4. introduce a vulnerable code change, triggering a PR scan which adds a comment to the Pull Request

This repository contains everything you need to complete the lab except for the prerequisites listed below.

# Prerequisites

1. [signup](https://github.com/signup) for a free GitHub Account
2. access to a Cloud Native Coverity (CNC) instance with scan farm

# Clone repository

1. Clone this repository into your GitHub account via _GitHub → New → Import a Repository_
   - enter https://github.com/blackduck-se/bd-integrations-lab3.git
   - enter repository name, e.g. hello-java
   - leave as public (required for GHAS on free accounts)

# Setup workflow

1. Confirm all GitHub Actions are allowed via _GitHub → Project → Settings → Actions → General → Actions Permissions_
2. Confirm GITHUB_TOKEN has workflow read & write permissions via _GitHub → Project → Settings → Actions → General → Workflow Permissions_
3. Add the following variables, adding COV_USER and COVERITY_PASSPHRASE as a **secret** via _GitHub → Project → Settings → Secrets and Variables → Actions_
   - COVERITY_URL
   - COV_USER
   - COVERITY_PASSPHRASE
4. Add a coverity.yaml to the project repository via _GitHub → Project → Add file → Create new file_

```
capture:
  build:
    clean-command: mvn -B clean
    build-command: mvn -B -DskipTests package
analyze:
  checkers:
    webapp-security:
      enabled: true
```

5. Create a new workflow via _GitHub → Project → Actions → New Workflow → Setup a workflow yourself_

```
# example workflow for Coverity scans using the Black Duck Security Scan Action
# https://github.com/marketplace/actions/black-duck-security-scan
name: Coverity
on:
  push:
    branches: [ main, master, develop, stage, release ]
  pull_request:
    branches: [ main, master, develop, stage, release ]
  workflow_dispatch:
jobs:
  coverity:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout Source
      uses: actions/checkout@v4
    - name: Setup Java JDK
      uses: actions/setup-java@v4
      with:
        java-version: 21
        distribution: temurin
        cache: maven
    - name: Coverity Scan
      uses: blackduck-inc/black-duck-security-scan@v2.1.1
      with:
        coverity_url: ${{ vars.COVERITY_URL }}
        coverity_user: ${{ secrets.COV_USER }}
        coverity_passphrase: ${{ secrets.COVERITY_PASSPHRASE }}
        coverity_policy_view: ${{ github.event_name != 'pull_request' && 'Outstanding Issues' || '' }}
        coverity_prComment_enabled: true
        github_token: ${{ secrets.GITHUB_TOKEN }}
        # coverity_local: true
        # include_diagnostics: true
#    - name: Save Logs
#      if: always()
#      uses: actions/upload-artifact@v4
#      with:
#        name: bridge-logs
#        path: ${{ github.workspace }}/.bridge
#        include-hidden-files: true
```

# Full Scan

1. Monitor your workflow run and wait for the scan to complete via _GitHub → Project → Actions → Coverity → Most recent workflow run → Coverity_ **Milestone 1** :heavy_check_mark:
2. Note that the workflow fails because of the "Outstanding Issues" [view](https://documentation.blackduck.com/bundle/coverity-docs/page/coverity-platform/topics/view_management.html) found some issues. **Milestone 2** :heavy_check_mark:
3. Click on the link in the console output to view the Outstanding Issues in Coverity Connect. **Milestone 3** :heavy_check_mark:

# PR scan

1. Add a new file CommandInjection.java to src/main/java via _GitHub → Project → Add file → Create new file_
```
import java.io.*;
import javax.servlet.http.HttpServletRequest;

public class CommandInjection {
    public static Process runCmd(HttpServletRequest request) throws IOException {
        String filename = request.getParameter("filename");
        ProcessBuilder builder = new ProcessBuilder("cat", filename);
        Process process = builder.start();
        return(process);
    }
}
```
2. Click on _Commit Changes_, select create a **new branch** and start a PR
3. Review changes and click on _Create Pull Request_
4. Monitor workflow run via _GitHub → Project → Actions → Coverity → Most recent workflow run → Coverity_
5. Once workflow completes, navigate back to the PR and review the PR comment via _GitHub → Project → Pull requests_ **Milestone 4** :heavy_check_mark:

# Congratulations

You have now configured a Coverity workflow in GitHub and demonstrated all the functionality of the [Black Duck Security Scan Action](https://github.com/marketplace/actions/black-duck-security-scan). :clap: :trophy:
