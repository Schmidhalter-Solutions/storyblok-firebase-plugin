# Storyblok Firebase Plugin


A Storyblok field plugin that provides a category selector with infinite scroll functionality. This plugin allows content editors to select multiple categories from a Firebase collection and store them in Storyblok.

## Features

- 🔄 Infinite scroll loading of categories
- ✅ Multi-select functionality
- 🔍 Search and filter capabilities
- 🎯 Automatic slug mapping
- 📱 Responsive design
- 🔒 Secure Firebase integration

## Installation

1. Clone the repository:
```bash
git clone git@github.com:Schmidhalter-Solutions/storyblok-firebase-plugin.git
```

2. Install dependencies:
```bash
npm install
```

3. Create a `.env` file in the root directory with your Firebase configuration:
```env
VITE_FIREBASE_API_KEY=your_api_key
VITE_FIREBASE_AUTH_DOMAIN=your_auth_domain
VITE_FIREBASE_PROJECT_ID=your_project_id
VITE_FIREBASE_STORAGE_BUCKET=your_storage_bucket
VITE_FIREBASE_MESSAGING_SENDER_ID=your_messaging_sender_id
VITE_FIREBASE_APP_ID=your_app_id
VITE_FIREBASE_COLLECTION=your_collection_name
```

## Development

To start the development server:

```bash
npm run dev
```

## Building

To build the plugin for production:

```bash
npm run build
```

## Usage in Storyblok

1. In your Storyblok space, go to Settings > Field Plugins
2. Add a new field plugin
3. Upload the built plugin files
4. Create a new field in your content type and select the category selector plugin

## Data Structure

The plugin expects categories in your Firebase collection to have the following structure:

```typescript
interface Category {
  id: string;
  name: {
    en: string;
  };
  order?: number;
  slug?: string;
}
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the LICENSE file for details. 
