# Wedding Website

A beautiful, feature-rich, and fully responsive wedding website with bilingual support (English/German).

## Features

1. **Fully Responsive Design** - Works seamlessly on desktop, tablet, and mobile devices
2. **Bilingual Support** - Switch between English and German languages
3. **RSVP System** - Direct integration with Google Sheets for guest responses
4. **Wishes Wall** - Guests can leave anonymous wishes and messages
5. **Memories Page** - Dedicated page for wedding memories and gallery
6. **Photo Gallery** - Display wedding photos with random photo viewer
7. **Event Details** - Complete schedule with ceremony, cocktail, banquet, and dance times
8. **Venue Information** - Interactive Google Maps with directions and shuttle information
9. **Dress Code & Menu** - Modal popups with detailed information
10. **Add to Calendar** - Support for multiple calendar systems
11. **Video Integration** - Embed YouTube videos for venue tours or highlights

## Pages

- **index.html** - Main wedding page with event details, venue info, and RSVP form
- **memories.html** - Memories page with wishes wall, gallery, and photo album

## Getting Started

1. Clone or download this repository
2. Open `index.html` in your web browser
3. Customize the content, images, and styling to match your wedding theme

## Customization

### Language Support
The website includes English and German translations. To add more languages or modify translations, edit the `translations` object in the JavaScript section of each HTML file.

### Google Apps Script Setup
For RSVP and Wishes Wall functionality, you'll need to set up Google Apps Script. See:
- `GOOGLE_APPS_SCRIPT_SETUP.md` - General setup instructions
- `WISHES_WALL_GOOGLE_APPS_SCRIPT.md` - Wishes wall specific setup

### Google Maps API
For venue map functionality, see `GOOGLE_MAPS_API_SETUP.md` for setup instructions.

## Technical Details

- Pure HTML, CSS, and JavaScript (jQuery)
- No backend server required - uses Google Sheets and Google Apps Script
- Hosted on GitHub Pages (or any static hosting service)
- Responsive design using Bootstrap and custom CSS

## Credits

Crafted with lots of ❤️ by Yi

Based on the original wedding website template.
