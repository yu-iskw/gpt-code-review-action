# gpt-code-review-action
A container GitHub Action to review a pull request by GPT.

If the size of a pull request is over the maximum chunk size of the OpenAI API, the Action will split the pull request into multiple chunks and generate review comments for each chunk.
And then the Action summarizes the review comments and posts a review comment to the pull request.

## Pre-requisites
We have to set a GitHub Actions secret `OPENAI_API_KEY` to use the OpenAI API so that we securely pass it to the Action.

## Inputs

- `openai_api_key`: The OpenAI API key to access the OpenAI API.
- `github_token`: The GitHub token to access the GitHub API.
- `github_repository`: The GitHub repository to post a review comment.
- `github_pull_request_number`: The GitHub pull request number to post a review comment.
- `git_commit_hash`: The git commit hash to post a review comment.
- `pull_request_diff`: The diff of the pull request to generate a review comment.
- `extra_prompt`: The extra prompt to generate a review comment.
- `model`: The model to generate a review comment.
- `temperature`: The temperature to generate a review comment.
- `top_p`: The top_p to generate a review comment.
- `max_tokens`: The max_tokens to generate a review comment.
- `frequency_penalty`: The frequency_penalty to generate a review comment.
- `presence_penalty`: The presence_penalty to generate a review comment.
- `log_level`: The log level to print logs.

## Example usage
Here is an example to use the Action to review a pull request of the repository.
The actual file is located at [`.github/workflows/test-action.yml`](.github/workflows/test-action.yml).
We set `extra_prompt` to `You are very familiar with python too.`, as the Action is implemented in Python.
We aim to make GPT review a pull request from a point of view of a Python developer.

As a result of an execution of the Action, the Action posts a review comment to the pull request like the following image.
![An example comment of the code review](./docs/images/example.png)

```yaml
name: "Test Code Review"

on:
  pull_request:
    paths-ignore:
      - "*.md"
      - "LICENSE"

jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/checkout@v3
      - name: "Get diff of the pull request"
        id: get_diff
        shell: bash
        env:
          PULL_REQUEST_HEAD_REF: "${{ github.event.pull_request.head.ref }}"
        run: |-
          git fetch origin "${{ env.PULL_REQUEST_HEAD_REF }}:${{ env.PULL_REQUEST_HEAD_REF }}"
          git checkout "${{ env.PULL_REQUEST_HEAD_REF }}"
          git diff "origin/${{ env.PULL_REQUEST_HEAD_REF }}" > "diff.txt"
          # shellcheck disable=SC2086
          echo "diff=$(cat "diff.txt")" >> $GITHUB_ENV
      - uses: yu-iskw/gpt-code-review-action@v0.1.0
        name: "Code Review by GPT"
        id: review
        with:
          openai_api_key: ${{ secrets.OPENAI_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          github_repository: ${{ github.repository }}
          github_pull_request_number: ${{ github.event.pull_request.number }}
          git_commit_hash: ${{ github.event.pull_request.head.sha }}
          model: "text-davinci-003"
          temperature: "0.7"
          max_tokens: "256"
          top_p: "1"
          frequency_penalty: "0.0"
          presence_penalty: "0.0"
          pull_request_diff: |-
            ${{ steps.get_diff.outputs.diff }}
          extra_prompt: |-
            You are very familiar with python too.
          log_level: "DEBUG"
```

## Known Issues
1. We may pay for the OpenAI API usage a lot, if we use the Action for a large number of pull requests.
2. The Action may fail due to the rate limit of the OpenAI API.
