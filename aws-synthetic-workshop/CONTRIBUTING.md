# Contributing to AWS Serverless Workshop

Thank you for your interest in contributing to this workshop! We welcome contributions from the community to make this resource better for everyone.

## 🤝 How to Contribute

### 1. Reporting Issues
- Use the [GitHub Issues](https://github.com/yourusername/aws-serverless-workshop/issues) page
- Search existing issues before creating a new one
- Provide detailed information about the problem
- Include steps to reproduce the issue

### 2. Suggesting Enhancements
- Open an issue with the "enhancement" label
- Clearly describe the proposed improvement
- Explain why this enhancement would be useful
- Consider the scope and complexity

### 3. Contributing Code
1. **Fork the repository**
2. **Create a feature branch**: `git checkout -b feature/your-feature-name`
3. **Make your changes**
4. **Test thoroughly**
5. **Commit with clear messages**: `git commit -m "Add: description of changes"`
6. **Push to your fork**: `git push origin feature/your-feature-name`
7. **Create a Pull Request**

## 📝 Content Guidelines

### Workshop Content
- **Accuracy**: Ensure all technical information is correct
- **Clarity**: Write clear, step-by-step instructions
- **Completeness**: Include all necessary details
- **Testing**: Verify all steps work as described

### Code Examples
- **Functionality**: All code must work as intended
- **Comments**: Include helpful comments
- **Best Practices**: Follow AWS and language-specific best practices
- **Security**: Never include sensitive information (keys, passwords, etc.)

### Documentation
- **Markdown**: Use proper Markdown formatting
- **Hugo**: Follow Hugo content structure
- **Bilingual**: Maintain both English and Vietnamese versions
- **Screenshots**: Update images when UI changes

## 🌐 Translation Guidelines

### Adding New Languages
1. Create new files with appropriate language suffix (e.g., `_index.es.md` for Spanish)
2. Translate all content accurately
3. Maintain the same structure and formatting
4. Update navigation and links

### Updating Existing Translations
- Keep translations synchronized with English content
- Maintain technical accuracy
- Preserve formatting and Hugo shortcodes

## 🔧 Technical Requirements

### Hugo Setup
```bash
# Install Hugo extended version
hugo version  # Should be v0.80.0 or later

# Test locally
hugo server -D
```

### File Structure
```
content/
├── 1-Introduce/
│   ├── _index.md      # English
│   └── _index.vi.md   # Vietnamese
├── 2-Prerequiste/
└── ...
```

### Formatting Standards
- Use **bold** for AWS services, buttons, and important terms
- Use `code blocks` for values, file names, and commands
- Use Hugo shortcodes for notices: `{{% notice info %}}`
- Include screenshots for complex UI steps

## 🧪 Testing Guidelines

### Before Submitting
- [ ] Test all workshop steps in a clean AWS account
- [ ] Verify all links work correctly
- [ ] Check Hugo site builds without errors
- [ ] Validate JSON and code syntax
- [ ] Ensure screenshots are up-to-date

### AWS Testing
- Use AWS Free Tier account
- Test in ap-southeast-1 region
- Verify cleanup procedures work
- Check cost implications

## 📋 Pull Request Process

### PR Requirements
1. **Clear title** describing the change
2. **Detailed description** of what was changed and why
3. **Testing notes** showing the changes were verified
4. **Screenshots** for UI-related changes
5. **Issue reference** if applicable

### Review Process
1. **Automated checks** must pass
2. **Manual review** by maintainers
3. **Testing verification** in AWS environment
4. **Approval** from at least one maintainer

## 🏷️ Commit Message Format

Use clear, descriptive commit messages:

```
Type: Brief description

Detailed explanation if needed

- Add: New feature or content
- Fix: Bug fix or correction
- Update: Modify existing content
- Remove: Delete content or files
- Docs: Documentation changes
```

Examples:
```
Add: DynamoDB table creation steps for Option 2

Fix: Correct Lambda function permissions in step 3.3

Update: Refresh screenshots for new AWS Console UI

Docs: Improve troubleshooting section in prerequisites
```

## 🎯 Areas for Contribution

### High Priority
- [ ] Update screenshots for latest AWS Console UI
- [ ] Add troubleshooting sections
- [ ] Improve error handling in code examples
- [ ] Add cost optimization tips

### Medium Priority
- [ ] Add more real-world use cases
- [ ] Create video tutorials
- [ ] Add advanced configuration options
- [ ] Improve mobile responsiveness

### Low Priority
- [ ] Add more language translations
- [ ] Create downloadable resources
- [ ] Add integration examples
- [ ] Expand monitoring section

## 🚫 What Not to Include

- **Sensitive Information**: AWS keys, passwords, personal data
- **Copyrighted Content**: Without proper attribution
- **Promotional Content**: Unrelated to the workshop
- **Incomplete Features**: Half-finished implementations
- **Breaking Changes**: Without proper migration guide

## 📞 Getting Help

If you need help with contributing:

- 💬 **GitHub Discussions**: Ask questions and get help
- 📧 **Email**: Contact maintainers directly
- 📖 **Documentation**: Check existing docs first
- 🐛 **Issues**: Search for similar problems

## 🏆 Recognition

Contributors will be:
- Listed in the README.md acknowledgments
- Credited in release notes for significant contributions
- Invited to become maintainers for consistent contributors

## 📄 Code of Conduct

By participating in this project, you agree to:
- Be respectful and inclusive
- Focus on constructive feedback
- Help create a welcoming environment
- Follow GitHub's Community Guidelines

Thank you for contributing to make this workshop better for everyone! 🙏