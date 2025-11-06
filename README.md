# LiB Digital Twin Cycling Aging Project

This repository contains Jupyter notebooks and data for lithium-ion battery capacity prediction and digital twin modeling.

## Repository Information

- **GitHub Repository**: <https://github.com/EigenJames/lib-digital-twin-cycling-aging>
- **Local Path**: /Volumes/WorkingData/Projects and doccuments/Project_Portfolio/LiB Digital_Twin cycling aging
- **Initial Commit**: 6f40a96 (Initial commit: LiB Digital Twin cycling aging project)
- **Files Committed**: 287 files, 207,556 lines added
- **Status**: Local git repository initialized and committed, GitHub repository created but push failed due to private email address

## Contents

- `discharge.csv`: Battery discharge data
- `LiB Capacity prediciotn.ipynb`: Notebook for capacity prediction (note: filename has typo)
- `LiB Capacity prediction readable.ipynb`: Readable version of capacity prediction notebook
- `first test.ipynb`: Initial test notebook
- `physical vs NN emperical.ipynb`: Comparison of physical vs neural network empirical models (note: filename has typo)
- `tuner_results/`: Hyperparameter tuning results for various models
  - `enhanced_lib_battery_tuning/`: Enhanced LiB battery tuning (30 trials)
  - `enhanced_residual_tuning/`: Enhanced residual tuning (25 trials)
  - `lib_battery_tuning/`: LiB battery tuning (20 trials)
  - `residual_tuning/`: Residual tuning (15 trials)

## Setup Instructions

1. Git repository has been initialized locally
2. GitHub repository has been created at <https://github.com/EigenJames/lib-digital-twin-cycling-aging>
3. To complete the push to GitHub:
   - Make your email address public in GitHub settings: <https://github.com/settings/emails>
   - Or set a public email in git config: `git config user.email "your-public-email@example.com"`
   - Then run: `git push -u origin master`

## Notes

- The repository contains large model checkpoint files (.h5 files) which may make the repository size significant
- Consider adding a .gitignore file to exclude unnecessary files in future commits
- The project appears to focus on machine learning models for battery capacity prediction using Keras/TensorFlow (based on .h5 checkpoint files)

## Git Commands Used

```bash
git init
git add .
git commit -m "Initial commit: LiB Digital Twin cycling aging project"
gh repo create lib-digital-twin-cycling-aging --public --source=. --remote=origin --push
```

## Next Steps

- Resolve email privacy issue and push to GitHub
- Add collaborators if needed
- Set up CI/CD if required
- Add proper documentation and requirements.txt for dependencies
