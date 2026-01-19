Implement all tasks assigned to you and ONLY those task(s) that have been assigned to you.

## Implementation process:

1. Check if deep reading is needed and perform if necessary:
   ```bash
   # Check abstraction layer distance
   {{workflows/scope-detection/calculate-abstraction-layer-distance}}
   
   # If deep reading is needed, perform deep reading
   {{workflows/deep-reading/read-implementation-deep}}
   
   # Detect reusable code
   if [ -f "$SPEC_PATH/implementation/cache/deep-reading/implementation-analysis.json" ]; then
       {{workflows/deep-reading/detect-reusable-code}}
   fi
   ```

2. Load basepoints knowledge (if available):
   ```bash
   # Determine spec path
   SPEC_PATH="geist/specs/[this-spec]"
   
   # Load extracted basepoints knowledge if available
   if [ -f "$SPEC_PATH/implementation/cache/basepoints-knowledge.json" ]; then
       EXTRACTED_KNOWLEDGE=$(cat "$SPEC_PATH/implementation/cache/basepoints-knowledge.json")
       SCOPE_DETECTION=$(cat "$SPEC_PATH/implementation/cache/scope-detection/semantic-analysis.json" 2>/dev/null || echo "{}")
       KEYWORD_MATCHING=$(cat "$SPEC_PATH/implementation/cache/scope-detection/keyword-matching.json" 2>/dev/null || echo "{}")
   fi
   ```

3. Load deep reading results (if available):
   ```bash
   if [ -f "$SPEC_PATH/implementation/cache/deep-reading/implementation-analysis.json" ]; then
       DEEP_READING_RESULTS=$(cat "$SPEC_PATH/implementation/cache/deep-reading/implementation-analysis.json")
   fi
   
   if [ -f "$SPEC_PATH/implementation/cache/deep-reading/reusable-code-detection.json" ]; then
       REUSABLE_CODE_RESULTS=$(cat "$SPEC_PATH/implementation/cache/deep-reading/reusable-code-detection.json")
   fi
   ```

4. Analyze the provided spec.md, requirements.md, and visuals (if any)
5. Analyze patterns in the codebase, basepoints knowledge (if available), AND deep reading results (if available) according to its built-in workflow
6. Use basepoints knowledge and deep reading results to:
   - Guide implementation with extracted patterns
   - Suggest reusable code and modules during implementation
   - Reference project-specific standards and coding patterns
   - Use extracted testing approaches for test writing
7. Use deep reading results to:
   - Guide implementation with actual implementation patterns
   - Suggest reusable code and modules from actual implementation
   - Reference similar logic found in implementation
   - Consider refactoring opportunities
8. Implement the assigned task group according to requirements, standards, basepoints knowledge, and deep reading results
9. Update `geist/specs/[this-spec]/tasks.md` to update the tasks you've implemented to mark that as done by updating their checkbox to checked state: `- [x]`

## Guide your implementation using:
- **The existing patterns** that you've found and analyzed in the codebase AND basepoints knowledge (if available) AND deep reading results (if available)
- **Basepoints knowledge** (if available):
  - Extracted patterns from basepoints
  - Reusable code and modules from basepoints
  - Project-specific standards and coding patterns from basepoints
  - Testing approaches from basepoints
- **Deep reading results** (if available):
  - Patterns extracted from actual implementation files
  - Similar logic found in implementation
  - Reusable code blocks, functions, and classes from implementation
  - Refactoring opportunities identified
  - Implementation analysis (logic flow, data flow, control flow, dependencies, patterns)
- **Specific notes provided in requirements.md, spec.md AND/OR tasks.md**
- **Visuals provided (if any)** which would be located in `geist/specs/[this-spec]/planning/visuals/`
- **User Standards & Preferences** which are defined below.

## Self-verify and test your work by:
- Running ONLY the tests you've written (if any) and ensuring those tests pass.
- IF your task involves user-facing UI, and IF you have access to browser testing tools, open a browser and use the feature you've implemented as if you are a user to ensure a user can use the feature in the intended way.
  - Take screenshots of the views and UI elements you've tested and store those in `geist/specs/[this-spec]/verification/screenshots/`.  Do not store screenshots anywhere else in the codebase other than this location.
  - Analyze the screenshot(s) you've taken to check them against your current requirements.
