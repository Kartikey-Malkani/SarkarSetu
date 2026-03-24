# Contributing to Sarkar Setu

Thank you for your interest in contributing to Sarkar Setu! We appreciate your efforts to help make this project better.

## Code of Conduct

This project and everyone participating in it is governed by our [Code of Conduct](CODE_OF_CONDUCT.md). By participating, you are expected to uphold this code.

## How Can I Contribute?

### Reporting Bugs

Before creating bug reports, please check the issue list as you might find out that you don't need to create one. When you are creating a bug report, please include as many details as possible:

* **Use a clear and descriptive title**
* **Describe the exact steps which reproduce the problem**
* **Provide specific examples to demonstrate the steps**
* **Describe the behavior you observed after following the steps**
* **Explain which behavior you expected to see instead and why**
* **Include screenshots and animated GIFs if possible**
* **Include your environment details** (OS, browser, Python version, etc.)

### Suggesting Enhancements

Enhancement suggestions are tracked as GitHub issues. When creating an enhancement suggestion, please include:

* **Use a clear and descriptive title**
* **Provide a step-by-step description of the suggested enhancement**
* **Provide specific examples to demonstrate the steps**
* **Describe the current behavior and expected behavior**
* **Explain why this enhancement would be useful**

### Pull Requests

* Fill in the required template
* Follow the Python/TypeScript styleguides
* Include appropriate test cases
* End all files with a newline
* Avoid platform-dependent code

## Styleguides

### Python Code Style

* Follow [PEP 8](https://www.python.org/dev/peps/pep-0008/)
* Use type hints for function arguments and return values
* Maximum line length: 100 characters
* Use meaningful variable names

```python
def recommend_schemes(user_profile: dict) -> list[dict]:
    """
    Recommend government schemes based on user profile.
    
    Args:
        user_profile: Dictionary containing user demographics
        
    Returns:
        List of recommended schemes sorted by relevance score
    """
    # Implementation here
```

### TypeScript/React Style

* Use functional components with hooks
* Use TypeScript interfaces for props
* Follow [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)
* Use meaningful component and variable names
* Add JSDoc comments for complex functions

```typescript
interface SchemeCardProps {
  scheme: Scheme;
  onSelect: (schemeId: string) => void;
}

export function SchemeCard({ scheme, onSelect }: SchemeCardProps) {
  return (
    // Component JSX
  );
}
```

### Commit Messages

* Use the present tense ("Add feature" not "Added feature")
* Use the imperative mood ("Move cursor to..." not "Moves cursor to...")
* Limit the first line to 72 characters or less
* Reference issues and pull requests liberally after the first line
* Use conventional commits format:
  * `feat:` - A new feature
  * `fix:` - A bug fix
  * `docs:` - Documentation only changes
  * `style:` - Changes that don't affect code meaning
  * `refactor:` - Code change that neither fixes a bug nor adds a feature
  * `perf:` - Code change that improves performance
  * `test:` - Adding missing tests or correcting existing tests

Examples:
```
feat(api): add endpoint for scheme recommendations
fix(frontend): resolve styling issue in dark mode
docs: update API documentation
```

## Development Setup

### Backend Development

```bash
cd sarkar_setu
python -m venv venv
source venv/bin/activate  # or `venv\Scripts\activate` on Windows
pip install -r requirements.txt
pip install -r requirements-dev.txt  # for development tools

# Set environment variables for local testing
export SIMULATE_PORTAL=false
export RUN_EMBEDDING_PIPELINE=false

# Run development server
python -m uvicorn src.api_server:app --reload
```

### Frontend Development

```bash
cd sarkar_setu/frontend
npm install
npm run dev

# Run linter
npm run lint

# Run tests
npm run test
```

## Testing

### Backend Tests

```bash
cd sarkar_setu
pytest tests/
pytest tests/test_api.py -v  # Run specific test file
```

### Frontend Tests

```bash
cd sarkar_setu/frontend
npm run test
npm run test -- --coverage  # With coverage report
```

## Documentation

When adding or modifying features, please update the relevant documentation:

* Update README.md if adding new features or changing setup instructions
* Update API documentation in `docs/API_OVERVIEW.md`
* Add docstrings to Python functions and classes
* Add JSDoc comments to TypeScript functions
* Update this file if adding new contribution guidelines

## Review Process

1. **Submit a Pull Request** with a clear description of your changes
2. **Code Review** - A maintainer will review your changes
3. **Feedback** - We may request changes or additional information
4. **Approval** - Once approved, your PR will be merged
5. **Release** - Your contribution will be included in the next release

## Community

* **Questions?** Create a new discussion in the GitHub Discussions tab
* **Found an issue?** Check existing issues first, then create a new one
* **Want to discuss changes?** Start a discussion before submitting a PR for significant changes

## Additional Notes

### Issue and Pull Request Labels

* `bug` - Something isn't working
* `enhancement` - New feature or request
* `documentation` - Improvements or additions to documentation
* `good first issue` - Good for newcomers
* `help wanted` - Extra attention is needed
* `question` - Further information is requested

### Recognition

Contributors will be recognized in:
* Project README
* GitHub contributors page
* Release notes (for significant contributions)

---

Thank you for contributing to Sarkar Setu! 🎉

For questions or concerns, please reach out to kartikeymalkani71@gmail.com
