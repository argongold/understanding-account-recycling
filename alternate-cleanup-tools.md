# AWS Account Cleanup Tools (Alternatives to aws-nuke)

Beyond **aws-nuke** (now maintained by [ekristen/aws-nuke](https://github.com/ekristen/aws-nuke), forked from the original rebuy-de/aws-nuke), here are other popular tools for AWS account cleanup across all enabled regions:

## Open-Source Tools

| Tool | Description | Language |
|------|-------------|----------|
| [**cloud-nuke**](https://github.com/gruntwork-io/cloud-nuke) (Gruntwork) | Deletes all resources in a cloud account. Supports filtering by region, resource type, and age. Actively maintained with 3k+ stars. | Go |
| [**aws-auto-cleanup**](https://github.com/mlevit/aws-auto-cleanup) | Programmatically deletes AWS resources based on an allowlist and TTL (time-to-live) settings. Good for scheduled/automated cleanup of sandbox accounts. | Python |
| [**aws-amicleaner**](https://github.com/bonclay7/aws-amicleaner) | Focused tool for cleaning up old/unused AMIs and their associated snapshots. | Python |

## AWS-Native Options

- **AWS Resource Explorer** — Not a deletion tool, but helps you discover and inventory resources across all regions before cleanup.
- **AWS Organizations cleanup strategies** — AWS provides [guidance for bulk account closure](https://aws.amazon.com/blogs/mt/streamlining-aws-organizations-cleanup-strategies/) when decommissioning entire accounts or organizations.
- **AWS Config + custom Lambda** — You can write rules that detect non-compliant or orphaned resources and trigger automated remediation/deletion.

## Key Differences

| Feature | aws-nuke | cloud-nuke | aws-auto-cleanup |
|---------|----------|------------|------------------|
| All regions | ✅ | ✅ | ✅ |
| Dry-run mode | ✅ | ✅ | ✅ |
| Filter by resource age | ❌ | ✅ | ✅ (TTL-based) |
| Scheduled/automated | Manual | Manual | Automated (Lambda) |
| Resource coverage | ~300+ types | ~80+ types | ~20+ types |
| Config-driven allowlist | ✅ | ✅ | ✅ |

## Recommendation

If you need broad coverage similar to aws-nuke, **cloud-nuke** from Gruntwork is the closest alternative — it's actively maintained, written in Go, and supports age-based filtering (e.g., "delete everything older than 24 hours"). For ongoing automated sandbox cleanup, **aws-auto-cleanup** is a good fit since it runs on a schedule via Lambda.

## References

- [gruntwork-io/cloud-nuke](https://github.com/gruntwork-io/cloud-nuke)
- [ekristen/aws-nuke](https://github.com/ekristen/aws-nuke)
- [mlevit/aws-auto-cleanup](https://github.com/mlevit/aws-auto-cleanup)
- [Streamlining AWS Organizations Cleanup Strategies](https://aws.amazon.com/blogs/mt/streamlining-aws-organizations-cleanup-strategies/)
- [bonclay7/aws-amicleaner](https://github.com/bonclay7/aws-amicleaner)
