# Generate pytest tests on a pull request

Label a trusted pull request `gen-tests` to get runnable pytest suggestions for changed Python files. The Action compares line and branch coverage with the existing tests, updates a single PR comment, and uploads the generated tests. It does not write to the PR branch.

## Try it without an API key

1. Copy [pytestgen.yml](pytestgen.yml) to `.github/workflows/pytestgen.yml` on your default branch. The example uses an Ubuntu runner and downloads a Qwen2.5-Coder 1.5B model for llama.cpp (about 1.1 GB). No GPU or paid API is needed.
2. Create the `gen-tests` label. Open a PR from a branch **in the same repository**, change a Python module under `src/`, and apply the label. Adjust `paths` if your packages live elsewhere; use `extra-packages` for dependencies needed to import your code.
3. Read the coverage comment and download **pytestgen-tests** from the linked workflow run. Review the expected values, then copy the useful cases into your own suite. A passing generated assertion can still preserve a bug.

## What the reviewer sees

In the [parcel-size example PR](https://github.com/ark2016/pytestgen-action-demo/pull/1), the existing test checked one small parcel. The PR introduced a new size above 10 kg. The [actual bot comment](https://github.com/ark2016/pytestgen-action-demo/pull/1#issuecomment-6915585513) includes a table like this:

| Changed file | Line coverage | Branch coverage | Tests kept |
| --- | --- | --- | ---: |
| `src/parcel_demo/sizes.py` | 50.0% → 100.0% | 33.3% → 100.0% | 8 |

All eight cases passed on the first run. Z3 supplied their inputs; the model reviewed the recorded results. This is one small example, not a prediction for other packages.

## Use your own model

Keep the checkout and Action steps in the starter workflow. Remove **Start a CPU model** and pass these inputs instead:

```yaml
      - uses: ark2016/mipt-9-pytestGen@v0.1.0
        with:
          llm-base-url: ${{ vars.PYTESTGEN_LLM_BASE_URL }}
          llm-model: ${{ vars.PYTESTGEN_LLM_MODEL }}
          llm-api-key: ${{ secrets.PYTESTGEN_LLM_API_KEY }}
```

The endpoint must be reachable from the runner and support OpenAI-compatible chat completions. An external provider receives excerpts of the code under test. The workflow executes project and generated Python code: run it only on changes you trust. The subprocess timeout is not a security boundary. Use a disposable runner and keep unrelated credentials out of the job.

See [all inputs and outputs](../../action.yml) for file limits, timeouts, project dependencies and reporting controls.
