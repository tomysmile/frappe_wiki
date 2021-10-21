## Pull Request Review Guidelines
Guidelines for anyone trying to review pull requests for Frappe Framework.

### Pull Request information
It is very important for a reviewer & author to make sure that the submitted pull request has enough information about the change they are trying to make via the pull request while they have the complete context of the change. This information can be used by other reviewers, contributors, and the QA team. It is also useful when someone wants to know why a specific change was done.
A pull request should have the following information so that it can be useful for others.
- Detailed rationale of why the change is required.
- In case of a bug fix 
   - Error traceback in textual format (instead of an image) so that it can be searched by other users facing the same problem
   - Link of the pull request that might have introduced the bug
- In case of UI related issue
   - An animated GIF or before/after screenshots so that others can quickly get the context for the change.

### Coding Standards

For a pull request to be accepted a reviewer should validate whether coding standards [mentioned here](https://github.com/frappe/erpnext/wiki/Coding-Standards) are followed by the PR author.

### Checklist for a Pull Request Review
- All CI checks are passed.
- Pull request has enough information about the changes
  - If it is a fix, check if the link to the PR which caused the issue is added.
  - Proper traceback of the issue in textual format and not image so that it can be searched.
- Check if [coding standards](https://github.com/frappe/erpnext/wiki/Coding-Standards) are met.
- Tests are added to avoid possible failures in the future mostly during other code changes and refactors
- Documentation added or updated accordingly based on the change made (if applicable)
- Once the PR has been reviewed and approved, ensure it is backported to older versions (if applicable)
- Make sure to merge any auto-generated backport PR once it has been validated for any compatibility issues