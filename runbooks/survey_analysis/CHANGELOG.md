# Changelog

All notable changes to the PatchFox Dataset Security Analysis Runbook.

---

## [2.2] - February 5, 2026

### Added
- **Step 9A:** Shadow Findings Analysis - Zero-day exposure window
  - Identifies vulnerabilities present in codebase BEFORE CVE publication
  - Calculates "shadow window" between package commit and CVE published_at
  - Shadow exposure distribution by severity
  - Package-level shadow risk ranking
  - Grading criteria for undetectable exposure periods
  - Critical insight: reveals true security debt from undetectable vulnerabilities

---

## [2.1] - January 20, 2026

### Added
- **Step 1:** API pagination support for large datasets (>1000 records)
  - Pagination scripts for metrics and edits fetching
  - Step 1.5 for critical CVE fetching
- **Step 8:** REST API alternative for when direct database access is unavailable
  - Database connection instructions
  - RPS-based approximation script

### Fixed
- **Temporal Utilities:** Divide-by-zero in `get_story_arc()` for datasets starting from zero
  - Now finds first non-zero value as baseline for meaningful percentage calculations
  - Added narrative handling for "grew from nothing" scenarios

### Changed
- Split monolithic runbook into modular documentation structure
- Created `/runbooks/survey_analysis/` directory with:
  - Main README
  - Individual step documents (12 files)
  - Reference documentation (metrics, utilities, grading, pitfalls)

---

## [2.0] - January 19, 2026

### Added
- Major refactor to story-driven temporal analysis
- Temporal Analysis Utilities section with Python functions:
  - `get_story_arc()` - Extract narrative arc
  - `identify_chapters()` - Break into distinct phases
  - `find_turning_points()` - Find trend changes
  - `find_notable_events()` - Find anomalies
  - `analyze_velocity()` - Calculate rate of change
  - `compare_periods()` - Compare periods
  - `tell_metric_story()` - Generate complete narrative
- Step 12 for report generation (terminal + markdown)
- Story-driven framing for all analysis steps

### Changed
- Refactored all steps to use narrative framing
- Each step now answers: where we started, what happened, key events, current state

---

## [1.0] - Initial Release

### Added
- Initial runbook structure
- Basic metric definitions
- Analysis methodology
- Grading criteria
- Checklist and quick reference
