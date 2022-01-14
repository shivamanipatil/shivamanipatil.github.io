---
title: "GSoC 2020 Report"
date : "2020-08-30"
draft: false
---

## Introduction

During my GSoC, I worked on **App Store improvements** project for **ns-3** organization. GSoC was my first programming experience outside personal projects and I thoroughly enjoyed the experience. I had an awesome opportunity to work for the ns-3 organization. My mentors [abhijithanilkumar](https://github.com/abhijithanilkumar/), [mishalshah](https://github.com/mishal23) and [adeepkit01](https://github.com/adeepkit01) were extremely responsive, helpful, and understanding. I would also like to thank [tomh](http://www.tomh.org/) sir - the ns-3 Organization Admin for his help and suggestions.

## Project outline

**Project Goals** : To develop a Jenkins server and add necessary updates to ns-3 AppStore to check on-demand if available, uploaded or updated apps/modules/forks to AppStore build and pass tests successfully for the given ns-3 versions and display that information on AppStore. And to improve the AppStore by addressing the existing issues.

The project was completed in the following phases :

1. **Community Bonding** - Solving some of the existing issues and getting familiar with the codebase.
2. **Phase 1** - Added build model to AppStore, experimented with Jenkins locally, and wrote bash scripts used by pipelines.
3. **Phase 2** - Added functionalities to AppStore for Jenkins communication, REST APIs, and their tests in AppStore for Jenkins, and Build history page for app release.
4. **Phase 3** - Added pipelines to deployed Jenkins servers, configured the environment for AppStore - Jenkins communication, and added contributing, installation, and Jenkins documentations.

## Work done

This contains the work done during various phases and links to docs/commits/merge requests generated.

### Community bonding

- [Doc](https://docs.google.com/document/d/1uw0aHN7BF-H9fR14gna_A-NMiEsdYxQQAfZ3s9YY5a0/edit?usp=sharing) : Database migration for ns-3 AppStore from SQLite to Mysql/Postgres.
- Participated in the coding sprint and following merge requests are made :
  - [#64](https://gitlab.com/nsnam/ns-3-AppStore/-/merge_requests/63) : Changed 'Link to tutorial' to 'Documentation Link'.
  - [#64](https://gitlab.com/nsnam/ns-3-AppStore/-/merge_requests/64) : Added user option to change password.
  - [#65](https://gitlab.com/nsnam/ns-3-AppStore/-/merge_requests/65) : Password-reset email's username and subject formatting improved.
- Reviewed existing dependencies for outdated and non-supported ones. Following merge requests were made to update the outdated dependencies :
  - [Doc](https://docs.google.com/document/d/1ylSdZ7zTM4MlSBqSHaik244cZsj_2nkffqPtxHMiQwc/edit?usp=sharing) Document showing dependency review.
  - [#66](https://gitlab.com/nsnam/ns-3-AppStore/-/merge_requests/66) : Removed django-admin-bootstrap.
  - [#67](https://gitlab.com/nsnam/ns-3-AppStore/-/merge_requests/67) : Django-environ replaced with python-dotenv.

### Phase 1

- [Commit](https://gitlab.com/shivamanipatil/ns-3-AppStore/-/commit/db059501e9537233da4f1294deb9d9039908df1d) : The Build model to store information about the build info for an app/module was created and test for the Build model was also added.
- Installed Jenkins on the local machine. Read Jenkins documentation about pipeline building and its terminologies. Explored bake for fetching and building ns-3/modules/forks.
- [Commit](https://gitlab.com/nsnam/ns-3-AppStore/-/commit/6c7c6a55bbe99dc952d920e2148106a8a096552b?merge_request_iid=69) : Added bash scripts which are used by Jenkins pipelines.

### Phase 2

- [Commit](https://gitlab.com/nsnam/ns-3-AppStore/-/commit/c39770666c14686b916880ad938a90a77d8987e1?merge_request_iid=69) : Functions to call Jenkins builds from appstore for all pipelines added.
- [Commit](https://gitlab.com/nsnam/ns-3-AppStore/-/commit/cb5637253f89e97497005eb62d41b025030f7d4a?merge_request_iid=69) : REST API to update the build model object from Jenkins added.
- [Commit](https://gitlab.com/nsnam/ns-3-AppStore/-/commit/bc93eff4c6302dc76664105599a707f4be11ca8d?merge_request_iid=69) : Added aborted and building statuses and changed date field to date-time.
- [Commit](https://gitlab.com/nsnam/ns-3-AppStore/-/commit/468ec37209f1b5f6685ce40fd8352c04ba1978dc?merge_request_iid=69) : Modified code so that build object is created before the build is called so that 'Building' status can be used.
- [Commit](https://gitlab.com/nsnam/ns-3-AppStore/-/commit/d695a210083a4a99f1d6364656d4820bc0105ec3?merge_request_iid=69) : Modify view and templates to display latest build info for a release.
- [Commit](https://gitlab.com/nsnam/ns-3-AppStore/-/merge_requests/69/diffs?commit_id=823cd8e332f0068da3449b7fea19d1387e7ea172) : Check if repo exists first then only check if a branch exists for it using GitPython.
- [Commit](https://gitlab.com/nsnam/ns-3-AppStore/-/commit/4966d962be544f73cc0c1ddff7da23ecc21c6566?merge_request_iid=69) : Authentication and permissions added for Update Build REST API endpoint.
- [Commit](https://gitlab.com/nsnam/ns-3-AppStore/-/commit/aada385072b22679a663ea38869d2ea6ed3ccda6?merge_request_iid=69) : Build history table for each release with pagination was added.
- [Commit](https://gitlab.com/nsnam/ns-3-AppStore/-/commit/a1bb41c3de9447256f16e9c1f94734ce682218c8?merge_request_iid=69) : Option to cancel the Jenkins builds from the appstore was added.
- [Commit](https://gitlab.com/nsnam/ns-3-AppStore/-/commit/561c65bb527d3606d21fe3424dc85109b5c74ccd?merge_request_iid=69) : Tests for PATCH REST API were added.

### Phase 3

- [Commit](https://gitlab.com/nsnam/ns-3-AppStore/-/commit/acfc4911043e3378d2173f0eea6bdd8343bf44ec?merge_request_iid=69) : Code refactoring. Comments/docstrings were added and unnecessary lines were removed.
- [Commit](https://gitlab.com/nsnam/ns-3-AppStore/-/commit/cef258add2b8204d4208f13c258e970bff0e3cae?merge_request_iid=69) : Jenkins environment variables were made to load from settings.
- [Commit](https://gitlab.com/nsnam/ns-3-AppStore/-/commit/2329f1d6b64a07e85ac666d07bd932b3f59298a5?merge_request_iid=69) : Exception handling for Jenkins build functions using try/except while covering corner cases.
- [Commit](https://gitlab.com/nsnam/ns-3-AppStore/-/commit/1a4934bc5880d4315d1e457379f45e04a9b6cb7c?merge_request_iid=69) : Static files for build templates were added and apps/tests.py was formated with pep8.
- [Commit](https://gitlab.com/nsnam/ns-3-AppStore/-/commit/96e0887e9d972d1e9648d35193460e400174b483?merge_request_iid=69) : Made Release model and forms enforce combined unique constraint for an app and version name.
- [Commit](https://gitlab.com/nsnam/ns-3-AppStore/-/commit/437d3ff46bb615e421d1212becd5e874a087de50?merge_request_iid=69) : REST API for Jenkins to call build on all releases using ns-3 dev.
- [Commit](https://gitlab.com/nsnam/ns-3-AppStore/-/commit/6e46b1587ff4a176a38e2d11c4b65c1822d46eb7?merge_request_iid=69) : Jenkinsfile's for all the pipelines were added.
- [Commit](https://gitlab.com/nsnam/ns-3-AppStore/-/commit/bbf603f40ccbae743296f72d6d03fd114dc43a42?merge_request_iid=69), [Commit](https://gitlab.com/nsnam/ns-3-AppStore/-/commit/d8563d60a1b790706def92b1b9073885417bae05?merge_request_iid=69) : Minor fixes.
- [Commit](https://gitlab.com/nsnam/ns-3-AppStore/-/commit/b142b7a891776cca704590cabbc58896004205f3?merge_request_iid=69) : README doc for setting the Jenkins pipeline.
- [Contributing guide](https://gitlab.com/shivamanipatil/ns-3-dev-fork/-/blob/new-docs/doc/contributing/source/app-store.rst) : Contributing guide for the AppStore - [hosted page link](https://shivamanipatil.github.io/contributing/build/html/app-store.html)  
- [Installation guide](https://gitlab.com/shivamanipatil/ns-3-dev-fork/-/blob/new-docs/doc/installation/source/app-store.rst) : Installation guide for the AppStore - [hosted page link](https://shivamanipatil.github.io/installation/build/html/app-store.html)
- All the pipelines and configurations were recreated on the deployed Jenkins server and tested with a dev appstore server.


## Other Miscellaneous Documents

- [Wiki - Weekly updates](https://www.nsnam.org/wiki/GSOC2020AppStore) : Weekly updates regarding the work.
- [Phase 1 Document](https://docs.google.com/document/d/1ekx4xlLK6KDj9TnFTpxVFp_7RelAYe7JvzX-Y5RRfhA/edit?usp=sharing) : Document explaining the work done during phase 1.
- [Work Guidelines](https://docs.google.com/document/d/19xdtI-qfJmVoJJK-JN88jyNBQAyrkBvfuM_nTfdSh_s/edit?usp=sharing) : Document explaining the scenarios supported by the Jenkins pipelines.
- [Phase 2 Document](https://docs.google.com/document/d/1KofTOsiA0I_HQFSQ_zc-iK2gdN2QRyYFoRiuG2GkIh4/edit?usp=sharing) : Document explaining the work done during phase 2.

## Future Work

- Jenkins Pipelines can be updated to handle more scenarios.
- Outstanding issues in the appstore can be solved.
- CI/CD pipelines can be refactored.

## Conclusion

I am happy that all the tasks proposed were completed. I want to thank all the people who motivated and helped me. I got to learn a lot during GSoC and couldn't have worked on a more fulfilling and exciting project. I will continue to contribute to open source and ns-3 in the future.
