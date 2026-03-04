# Smart Hebrew Recipe Search - מתכונים חכמים

## Overview
A Hebrew RTL recipe search engine web application. Only admins can add/edit/delete recipes, while users can browse, search, and view them. Features smart recipe suggestions, quick filters, complete admin management system, structured recipe data with image galleries, step-by-step instructions, PWA support for mobile installation, and user-facing features: database-backed favorites, shopping list generator, weekly meal planner, advanced search filters, voice search, cooking mode, and ingredient-based recipe discovery. Includes rating system (1-10 per recipe with averages), recipe author tracking, and author-based filtering.

## Recent Changes
- **2026-03-02**: Added rating system (1-10 per recipe, average display on cards/detail, per-user ratings), recipe author name field (required on create, shown on cards/detail), filter by author dropdown in advanced filters
- **2026-02-23**: Admin recipe form ingredient input upgraded to autocomplete from ingredients_master with "Choose from list" category modal, non-kosher ingredient validation (shrimp/lobster/crab/etc blocked), removed shrimp from DB + seed data, added DB index on ingredients_master(name_hebrew)
- **2026-02-22**: Weekly meal planner upgraded to support multiple recipes per day with add/remove per item, favorites count displayed on recipe cards and recipe detail page with optimistic updates
- **2026-02-22**: Comprehensive ingredient upgrade: 300+ Hebrew kosher ingredients seeded across 15 categories, numbered meat cuts (1-19 kosher), subcategory support, display_order, admin category filter view, "Add new ingredient" from pantry screen, public ingredient creation endpoint
- **2026-02-22**: Upgraded "I Have Ingredients" to categorized ingredient management system: ingredient_categories table, ingredients_master table, user_ingredients table, admin CRUD for categories + ingredients, categorized collapsible picker with search, DB-backed user ingredient selections, recipe matching by ingredient IDs with ranking
- **2026-02-22**: Added user features: DB-backed favorites (per-device UUID), shopping list, weekly meal planner, advanced filters (time/difficulty/origin), voice search (Web Speech API), cooking mode (fullscreen step-by-step with TTS), "I Have Ingredients" picker, favorites heart + shopping list on recipe details
- **2026-02-21**: Added recipe card image carousel (prev/next with dot indicators), both prep and total time display, favorites with localStorage persistence
- **2026-02-21**: Migrated image uploads to Supabase Storage (bucket: recipes-images, folder: recipes, public URLs)
- **2026-02-19**: Added PWA support (manifest.json, service worker, app icons, meta tags)
- **2026-02-19**: Enhanced recipe details page with image gallery slider, structured ingredients with checkboxes, step visualization with per-step images
- **2026-02-19**: Added native Web Share API integration (with clipboard fallback) for recipe sharing
- **2026-02-19**: Added normalized tables: recipe_images (gallery), recipe_steps (structured instructions), recipe_ingredients (structured with quantity/unit)
- **2026-02-19**: Added recipe metadata: preparationTime, totalTime, servings, difficulty, origin
- **2026-02-19**: Updated admin recipe form with gallery upload, structured ingredients, step builder
- **2026-02-19**: Updated recipe cards to show time, servings, difficulty, origin
- **2026-02-19**: Replaced Replit Auth with custom admin authentication (username/password with bcrypt)
- **2026-02-19**: Added admins table with roles (super_admin, admin), active/inactive status, force password change
- **2026-02-19**: Added image upload support (file upload + camera capture) stored in /uploads/recipes/
- **2026-02-19**: Built admin management UI (create/delete/toggle active admins) - super_admin only
- **2026-02-19**: Recipe CRUD with admin tracking (createdByAdminId), tags, days, holidays
- **2026-02-19**: Ingredient-based search in addition to title/description search

## Architecture
- **Frontend**: React + Vite + TanStack Query + wouter + Tailwind CSS + shadcn/ui + Framer Motion
- **Backend**: Express.js + Drizzle ORM + PostgreSQL
- **Auth**: Custom session-based auth with bcrypt password hashing, express-session + connect-pg-simple
- **Image Upload**: Multer + Supabase Storage (bucket: recipes-images, folder: recipes, public URLs)
- **PWA**: manifest.json + service worker (network-first with cache fallback)

## Database Schema
- `admins` - id, username, password(hashed), role(super_admin/admin), active, force_password_change, created_at
- `recipes` - id, title, description, ingredients(jsonb), preparation(jsonb), image_url, category, tags[], suitable_days[], suitable_holidays[], created_by_admin_id, preparation_time, total_time, servings, difficulty, origin, author_name, created_at, updated_at
- `recipe_images` - id, recipe_id, image_path, display_order (gallery images)
- `recipe_steps` - id, recipe_id, step_number, step_text, step_image_path (structured steps)
- `recipe_ingredients` - id, recipe_id, ingredient_name, quantity, unit, ingredient_id (structured ingredients, linked to master)
- `ingredient_categories` - id, name_hebrew (unique, e.g. ירקות, פירות, בשר)
- `ingredients_master` - id, name_hebrew (unique), category_id (FK to ingredient_categories), subcategory (text, nullable), display_order (int)
- `user_ingredients` - id, user_id, ingredient_id (unique per user+ingredient, device-based selection)
- `user_favorites` - id, user_id, recipe_id, created_at (device UUID favorites)
- `shopping_list_items` - id, user_id, ingredient_name, quantity, unit, recipe_title, completed, created_at
- `recipe_ratings` - id, user_id, recipe_id, rating (1-10), created_at (unique per user+recipe)
- `meal_plans` - id, user_id, day_of_week, recipe_id, created_at (weekly planner)
- `admin_sessions` - session store for admin auth

## User Identification
- Device-based UUID stored in localStorage (no login required)
- Sent via `x-user-id` header on all user API requests
- `userFetch()` utility in `client/src/lib/user-id.ts` auto-adds the header

## Type System
- `RecipeWithAdmin` - Recipe + createdByAdmin username + averageRating + ratingCount (for list views)
- `RecipeFull` - RecipeWithAdmin + images[] + steps[] + structuredIngredients[] + userRating (for detail view)
- Backward compatibility: legacy jsonb fields (ingredients, preparation) still populated alongside normalized tables
- Constants: DIFFICULTY_OPTIONS, ORIGIN_OPTIONS, UNIT_OPTIONS exported from schema

## Default Admin
- Username: `admin`, Password: `1234` (must change on first login)
- Role: super_admin

## Key Routes
- `/` - Home page with search, quick filters, advanced filters, voice search, ingredient picker, recipe grid
- `/recipe/:id` - Recipe details with gallery, cooking mode, favorites heart, shopping list add, share
- `/favorites` - User's favorited recipes (per-device)
- `/shopping-list` - Shopping list with toggle completion, remove, clear
- `/meal-planner` - Weekly meal planner (assign recipes to days)
- `/auth` - Admin login page
- `/admin` - Admin dashboard (recipes list, add/edit recipe, admin management)
- `/change-password` - Change password page

## API Endpoints
- `GET /api/recipes` - List recipes (search, category, tag, difficulty, origin, maxPrepTime, maxTotalTime, ingredients, ingredientIds)
- `GET /api/ingredients` - List master ingredients (search, categoryId query params)
- `POST /api/ingredients` - Add new ingredient (public, for pantry screen)
- `GET /api/ingredient-categories` - List categories with their ingredients
- `POST /api/admin/ingredient-categories` - Create category (admin only)
- `PUT /api/admin/ingredient-categories/:id` - Update category (admin only)
- `DELETE /api/admin/ingredient-categories/:id` - Delete category (admin only)
- `POST /api/admin/ingredients` - Create master ingredient (admin only)
- `PUT /api/admin/ingredients/:id` - Update master ingredient (admin only)
- `DELETE /api/admin/ingredients/:id` - Delete master ingredient (admin only)
- `GET /api/user/ingredients` - Get user's selected ingredient IDs
- `POST /api/user/ingredients` - Set user's ingredient selections (batch)
- `POST /api/user/ingredients/:ingredientId/toggle` - Toggle single ingredient
- `GET /api/user/favorites` - Get user's favorite recipe IDs
- `GET /api/user/favorites/recipes` - Get user's favorite recipes with details
- `POST /api/user/favorites/:recipeId` - Add favorite
- `DELETE /api/user/favorites/:recipeId` - Remove favorite
- `GET /api/user/shopping-list` - Get shopping list items
- `POST /api/user/shopping-list` - Add single item
- `POST /api/user/shopping-list/batch` - Add multiple items (from recipe)
- `PATCH /api/user/shopping-list/:id/toggle` - Toggle item completion
- `DELETE /api/user/shopping-list/:id` - Remove item
- `DELETE /api/user/shopping-list` - Clear entire list
- `GET /api/favorites/counts` - Get global favorite counts for all recipes
- `GET /api/favorites/count/:recipeId` - Get favorite count for single recipe
- `GET /api/ratings` - Get all recipe average ratings
- `GET /api/ratings/:recipeId` - Get average rating for single recipe
- `GET /api/user/rating/:recipeId` - Get user's rating for a recipe
- `POST /api/user/rating/:recipeId` - Rate a recipe (1-10)
- `GET /api/authors` - Get list of unique recipe authors
- `GET /api/user/meal-plans` - Get weekly meal plans (multiple per day)
- `POST /api/user/meal-plans` - Add recipe to a day (supports multiple)
- `DELETE /api/user/meal-plans/item/:id` - Remove single meal plan item by ID
- `DELETE /api/user/meal-plans/:day` - Remove all meal plans for a day
- `GET /api/recipes/:id` - Get full recipe (includes images, steps, structuredIngredients)
- `POST /api/recipes` - Create recipe with nested data (admin only)
- `PUT /api/recipes/:id` - Update recipe with nested data (admin only, own or super_admin)
- `DELETE /api/recipes/:id` - Delete recipe (admin only, own or super_admin)
- `GET /api/recipes/suggest` - Get smart recipe suggestion
- `POST /api/admin/login` - Admin login
- `POST /api/admin/logout` - Admin logout
- `GET /api/admin/me` - Get current admin
- `POST /api/admin/change-password` - Change password
- `GET /api/admins` - List all admins (super_admin only)
- `POST /api/admins` - Create admin (super_admin only)
- `PUT /api/admins/:id` - Update admin (super_admin only)
- `DELETE /api/admins/:id` - Delete admin (super_admin only)
- `PATCH /api/admins/:id/toggle-active` - Toggle admin active status (super_admin only)
- `POST /api/upload/image` - Upload single recipe image (admin only)
- `POST /api/upload/images` - Upload multiple recipe images (admin only)

## User Preferences
- Hebrew RTL interface throughout
- Mobile-first responsive design
- Rubik font for Hebrew text
- Warm orange/amber color scheme
