# Speech Therapy Waiting List | רשימת המתנה לקלינאות תקשורת

A bilingual (Hebrew/English) waiting list form for speech therapy services with a modern, accessible design.

## 🌟 Features

- **Bilingual Support**: Full Hebrew and English translation with RTL support
- **Responsive Design**: Works seamlessly on mobile, tablet, and desktop devices
- **Accessible**: WCAG-compliant with screen reader support and keyboard navigation
- **Multi-Step Form**: Progressive form with 5 steps for better user experience
- **Auto-Save**: Automatically saves form progress to prevent data loss
- **Form Validation**: Client-side validation with helpful error messages
- **Modern UI**: Clean design with Material Icons and Google Fonts

## 📋 Form Steps

1. **Patient Information**: Name and relationship to patient
2. **Contact Information**: Phone number and email address
3. **Concerns**: Age group and description of speech/communication challenges
4. **Availability**: Time slot selection for appointments
5. **Review & Submit**: Final review and consent

## 🚀 Getting Started

### Prerequisites

- A form backend service for collecting submissions:
  - **Recommended**: Google Sheets (see [Google Sheets Integration Guide](GOOGLE_SHEETS_INTEGRATION.md))
  - **Alternative**: Formspree account
- A web server to host the static HTML file

### Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/ikorecky-com/ikorecky-waiting-list.git
   ```

2. Set up your form backend:
   
   **Option A: Google Sheets Integration (Recommended)**
   - Follow the comprehensive [Google Sheets Integration Guide](GOOGLE_SHEETS_INTEGRATION.md)
   - Provides direct data access, real-time updates, and built-in analysis tools
   
   **Option B: Formspree Integration**
   - Update the Formspree form ID in `index.html`:
     ```javascript
     // Line ~2196
     const response = await fetch('https://formspree.io/f/YOUR_FORM_ID', {
     ```
     Replace `YOUR_FORM_ID` with your actual Formspree form ID.

3. Deploy to your web server or use GitHub Pages.

## 🛠️ Customization

### Changing Colors

The design system uses CSS custom properties defined in `:root`:

```css
:root {
    --color-primary: #2A9D8F;        /* Main teal color */
    --color-secondary: #E9967A;      /* Coral accent */
    --color-success: #1E7E34;        /* Success green */
    --color-error: #C41E3A;          /* Error red */
}
```

### Adding Translations

Translations are defined in the `translations` object (lines ~1598-1787):

```javascript
const translations = {
    en: { /* English translations */ },
    he: { /* Hebrew translations */ }
};
```

### Modifying Form Fields

Form fields can be customized by editing the respective step sections in the HTML.

## 📱 Browser Support

- Chrome (latest)
- Firefox (latest)
- Safari (latest)
- Edge (latest)
- Mobile browsers (iOS Safari, Chrome Mobile)

## 🔒 Privacy & Security

- No data is stored in the browser beyond auto-save functionality
- Auto-save data is cleared after 7 days or upon successful submission
- All form data is transmitted securely via HTTPS to Formspree
- No third-party tracking or analytics

## 📄 License

This project is open source and available under the MIT License.

## 👥 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 🐛 Reporting Issues

If you find a bug or have a suggestion, please open an issue on GitHub.

## 📞 Contact

For questions or support, please contact through the website at [ikorecky.com](https://ikorecky.com)

## 🙏 Acknowledgments

- Material Symbols by Google
- Google Fonts (Assistant, Inter)
- Formspree for form handling
