name it `gh_my_pr.1h.sh`


```bash
#!/bin/bash

# <xbar.title>GitHub PR Dashboard</xbar.title>
# <xbar.version>v1.1</xbar.version>
# <xbar.author>Giorgio Vilardo</xbar.author>
# <xbar.desc>Shows GitHub PRs awaiting your review and assigned to you</xbar.desc>
# <xbar.dependencies>gh cli, jq</xbar.dependencies>

# Check dependencies
if ! command -v gh &>/dev/null || ! command -v jq &>/dev/null; then
    echo "GitHub ❌"
    echo "---"
    echo "Error: GitHub CLI or jq not available"
    exit 1
fi

# Fetch PRs awaiting review
review_prs=$(GH_PAGER= gh search prs --review-requested=@me --state=open --json url,title,labels 2>/dev/null)
review_count=0
if [[ $? -eq 0 && -n "$review_prs" ]]; then
    review_count=$(echo "$review_prs" | jq '. | length' 2>/dev/null || echo 0)
fi

# Fetch assigned PRs
assigned_prs=$(GH_PAGER= gh search prs --assignee=@me --state=open --json url,title,labels 2>/dev/null)
assigned_count=0
if [[ $? -eq 0 && -n "$assigned_prs" ]]; then
    assigned_count=$(echo "$assigned_prs" | jq '. | length' 2>/dev/null || echo 0)
fi

# Calculate total
total_count=$((review_count + assigned_count))

# Display count in menu bar
if [ "$total_count" -eq 0 ]; then
    echo "GitHub (0)"
else
    echo "GitHub ($total_count)"
fi

echo "---"

# Display PRs to review
if [ "$review_count" -eq 0 ]; then
    echo "No PRs awaiting review | bash=/bin/true terminal=false"
else
    echo "PRs awaiting review ($review_count): | bash=/bin/true terminal=false"
    echo "---"
    echo "$review_prs" | jq -r '.[] |
        "\(.title)" +
        (if (.labels | length) > 0 then " [" + (.labels | map(.name) | join(", ")) + "]" else "" end) +
        " | href=\(.url) tag=review"'
fi

# Add separator if both sections have content
if [ "$review_count" -gt 0 ] && [ "$assigned_count" -gt 0 ]; then
    echo "---"
fi

# Display assigned PRs
if [ "$assigned_count" -eq 0 ]; then
    echo "No assigned PRs | bash=/bin/true terminal=false"
else
    echo "Assigned PRs ($assigned_count): | bash=/bin/true terminal=false"
    echo "---"
    echo "$assigned_prs" | jq -r '.[] |
        "\(.title)" +
        (if (.labels | length) > 0 then " [" + (.labels | map(.name) | join(", ")) + "]" else "" end) +
        " | href=\(.url) tag=assigned"'
fi

echo "---"
echo "Refresh | refresh=true"
```
