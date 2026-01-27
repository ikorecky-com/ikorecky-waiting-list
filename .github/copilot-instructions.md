# GitHub Copilot Instructions

## Project Overview
This is a bilingual (Hebrew/English) static waiting list website for speech therapy services. The site is hosted on GitHub Pages and uses a single-page HTML application with embedded CSS and JavaScript.

## Tech Stack
- **Frontend**: Pure HTML5, CSS3, and vanilla JavaScript (no frameworks)
- **Fonts**: Google Fonts (Assistant for Hebrew, Inter for English)
- **Icons**: Material Symbols
- **Hosting**: GitHub Pages (served via CNAME: ikorecky.com)

## Code Structure
- `index.html`: Main application file containing all HTML, CSS, and JavaScript
- `CNAME`: Domain configuration for GitHub Pages

## Code Guidelines

### HTML
- Use semantic HTML5 elements
- Maintain bilingual support (Hebrew RTL and English LTR)
- Keep accessibility in mind (ARIA labels, semantic structure)

### CSS
- Follow the existing CSS Design System defined in `:root` variables
- Use CSS custom properties (variables) for colors and spacing
- Maintain responsive design principles
- Keep RTL (right-to-left) support for Hebrew content

### JavaScript
- Use vanilla JavaScript (no frameworks)
- Follow existing patterns for bilingual functionality
- Keep code embedded in the HTML file for simplicity

### Styling Conventions
- Color palette:
  - Primary (Teal): `--color-primary`, `--color-primary-hover`, `--color-primary-light`, `--color-primary-dark`
  - Secondary (Coral): `--color-secondary`, `--color-secondary-hover`, `--color-secondary-light`
  - Semantic colors for success, error, and warning states
- Use consistent spacing and typography defined in CSS variables

### Content Guidelines
- All user-facing text must be bilingual (Hebrew and English)
- Respect cultural sensitivities for both language audiences
- Maintain professional tone appropriate for healthcare services

## Important Considerations
- **Do not** add build tools or compilation steps - this is a static site
- **Do not** introduce JavaScript frameworks or libraries unless absolutely necessary
- **Do not** separate CSS/JS into external files - keep the single-file architecture
- **Always** test changes in both Hebrew and English language modes
- **Always** test responsive behavior across different screen sizes

## Deployment
This site is automatically deployed via GitHub Pages from the main branch. Any changes to `index.html` are immediately live.
