# How to Build a Custom `DynamicLowerBar` Navigation Widget in FlutterFlow

Building a single custom bottom navigation widget (`DynamicLowerBar`) in FlutterFlow gives your e-commerce app a centralized, state-aware navigation bar that adapts dynamically to guest vs. logged-in sessions and displays live item-count badges without duplicating UI across screens.

---

## Why Use a Custom `DynamicLowerBar` Instead of FlutterFlow's Built-In or Conditional-Stack NavBar?

* **Single Source of Truth Across Every Page:** With FlutterFlow's native NavBar or manually stacked conditional rows, adding a new tab, tweaking icon spacing, or changing a route forces you to edit every page individually. A custom widget centralizes all layout, styling, and routing logic in one file.
* **True Dynamic Role & Guest Switching:** Native bottom bars struggle to swap tabs cleanly based on session state. Watching `FFAppState()` directly inside a custom widget lets the final tab switch automatically between **Profile** (for authenticated users) and **Sign In** (for guests), while resetting guest flags cleanly on tap.
* **Live Reactive Badges Without Re-Querying:** Because the widget subscribes to `FFAppState()` via `context.watch<FFAppState>()`, live badges—such as your wishlist item count—update instantaneously across every page the moment a user taps a heart icon.
* **Zero Duplicate-Tab Navigation Loops:** Passing a `currentPage` parameter prevents the widget from pushing duplicate routes onto the navigation stack when a user taps the tab they are already viewing.
* **Consistent Sizing & Overflow Protection:** Wrapping 5 e-commerce tabs (`Home`, `Categories`, `Wishlist`, `My Cart`, `Profile`) inside a single custom `Row` with controlled padding and `TextOverflow.ellipsis` guarantees consistent alignment across narrow and wide phone screens without layout jitter.

---

## Step 1: Prepare Required App State Variables & Page Routes

Before creating the widget, ensure the following exist in your FlutterFlow project:

1. **App State Variables** *(under **App Values → App State**)*:
* `isGuest` (`Boolean`, default: `false`) — tracks whether the user is browsing in guest mode.
* `wishlistProductIds` (`List<String>`, persisted: `true`) — stores saved product IDs for the live badge count.


2. **Target Page Route Names** *(check each page's **Route Settings** in FlutterFlow)*:
* `HomeDashboard`
* `CategoryExplorer`
* `WishlistProducts`
* `CartPage`
* `CustomerProfile`
* `MainLoginPage`



---

## Step 2: Create the `DynamicLowerBar` Custom Widget

1. Open **Custom Code** (the `</>` icon in the left sidebar) and click **+ Add** → **Widget**.
2. Set the **Widget Name** to `DynamicLowerBar`.
3. In the right-hand **Define Parameters** panel, add:
* **Parameter Name:** `currentPage`
* **Type:** `String`
* **Nullable:** Unchecked *(Required)*
* *(Leave `width` and `height` as FlutterFlow's default optional widget dimensions.)*


4. Click **Save** to generate the boilerplate imports.

---

## Step 3: Add the Complete `DynamicLowerBar` Code

Paste the following code into the editor (keeping FlutterFlow's automatic imports at the top) and click **Compile** to verify there are zero errors:

```dart
// Automatic FlutterFlow imports
import '/backend/backend.dart';
import '/flutter_flow/ff_builtin_enums.dart';
import '/flutter_flow/flutter_flow_theme.dart';
import '/flutter_flow/flutter_flow_util.dart';
import '/custom_code/widgets/index.dart'; // Imports other custom widgets
import '/custom_code/actions/index.dart'; // Imports custom actions
import '/flutter_flow/custom_functions.dart'; // Imports custom functions
import 'package:flutter/material.dart' hide RepeatMode;
// Begin custom widget code
// DO NOT REMOVE OR MODIFY THE CODE ABOVE!

import 'package:google_fonts/google_fonts.dart';
import 'package:provider/provider.dart';

class DynamicLowerBar extends StatefulWidget {
  const DynamicLowerBar({
    super.key,
    this.width,
    this.height,
    required this.currentPage, // 'home' | 'categories' | 'wishlist' | 'cart' | 'profile' | 'login'
  });

  final double? width;
  final double? height;
  final String currentPage;

  @override
  State<DynamicLowerBar> createState() => _DynamicLowerBarState();
}

class _DynamicLowerBarState extends State<DynamicLowerBar> {
  Widget _buildNavItem({
    required IconData activeIcon,
    required IconData inactiveIcon,
    required String label,
    required bool isSelected,
    required VoidCallback onTap,
    int badgeCount = 0,
  }) {
    final activeColor = FlutterFlowTheme.of(context).primary;
    final inactiveColor = FlutterFlowTheme.of(context).secondaryText;

    return InkWell(
      splashColor: Colors.transparent,
      focusColor: Colors.transparent,
      hoverColor: Colors.transparent,
      highlightColor: Colors.transparent,
      onTap: onTap,
      child: Column(
        mainAxisSize: MainAxisSize.min,
        mainAxisAlignment: MainAxisAlignment.center,
        crossAxisAlignment: CrossAxisAlignment.center,
        children: [
          Stack(
            clipBehavior: Clip.none,
            children: [
              Icon(
                isSelected ? activeIcon : inactiveIcon,
                color: isSelected ? activeColor : inactiveColor,
                size: 24.0,
              ),
              if (badgeCount > 0)
                Positioned(
                  right: -8.0,
                  top: -4.0,
                  child: Container(
                    padding: const EdgeInsets.symmetric(
                      horizontal: 4.5,
                      vertical: 1.5,
                    ),
                    decoration: BoxDecoration(
                      color: const Color(0xFFEF4444),
                      borderRadius: BorderRadius.circular(10.0),
                      border: Border.all(
                        color: FlutterFlowTheme.of(context).secondaryBackground,
                        width: 1.5,
                      ),
                    ),
                    constraints: const BoxConstraints(
                      minWidth: 16.0,
                      minHeight: 16.0,
                    ),
                    child: Center(
                      child: Text(
                        badgeCount > 99 ? '99+' : '$badgeCount',
                        style: GoogleFonts.inter(
                          color: Colors.white,
                          fontSize: 9.5,
                          fontWeight: FontWeight.w700,
                          height: 1.0,
                        ),
                      ),
                    ),
                  ),
                ),
            ],
          ),
          const SizedBox(height: 4.0),
          Text(
            label,
            maxLines: 1,
            overflow: TextOverflow.ellipsis,
            style: FlutterFlowTheme.of(context).labelSmall.override(
                  font: GoogleFonts.inter(
                    fontWeight: isSelected ? FontWeight.w700 : FontWeight.w500,
                  ),
                  color: isSelected ? activeColor : inactiveColor,
                  fontSize: 11.0,
                  letterSpacing: 0.0,
                  lineHeight: 1.2,
                ),
          ),
        ],
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    bool isGuest = false;
    int wishlistCount = 0;

    try {
      context.watch<FFAppState>();
      isGuest = FFAppState().isGuest;
      wishlistCount = FFAppState().wishlistProductIds.length;
    } catch (_) {
      isGuest = false;
      wishlistCount = 0;
    }

    return Container(
      width: widget.width ?? MediaQuery.sizeOf(context).width,
      height: widget.height ?? 80.0,
      decoration: BoxDecoration(
        color: FlutterFlowTheme.of(context).secondaryBackground,
        shape: BoxShape.rectangle,
      ),
      child: Column(
        mainAxisSize: MainAxisSize.min,
        crossAxisAlignment: CrossAxisAlignment.stretch,
        children: [
          Container(
            height: 1.0,
            decoration: BoxDecoration(
              color: FlutterFlowTheme.of(context).alternate,
              shape: BoxShape.rectangle,
            ),
          ),
          Expanded(
            child: Padding(
              padding:
                  const EdgeInsetsDirectional.fromSTEB(12.0, 0.0, 12.0, 0.0),
              child: Row(
                mainAxisSize: MainAxisSize.max,
                mainAxisAlignment: MainAxisAlignment.spaceAround,
                crossAxisAlignment: CrossAxisAlignment.center,
                children: [
                  // 1. HOME TAB
                  _buildNavItem(
                    activeIcon: Icons.home_rounded,
                    inactiveIcon: Icons.home_outlined,
                    label: 'Home',
                    isSelected: widget.currentPage == 'home',
                    onTap: () async {
                      if (widget.currentPage != 'home') {
                        try {
                          context.goNamed('HomeDashboard');
                        } catch (_) {}
                      }
                    },
                  ),

                  // 2. CATEGORIES TAB
                  _buildNavItem(
                    activeIcon: Icons.grid_view_rounded,
                    inactiveIcon: Icons.grid_view_outlined,
                    label: 'Categories',
                    isSelected: widget.currentPage == 'categories',
                    onTap: () async {
                      if (widget.currentPage != 'categories') {
                        try {
                          context.goNamed('CategoryExplorer');
                        } catch (_) {}
                      }
                    },
                  ),

                  // 3. WISHLIST TAB (WITH LIVE COUNT BADGE)
                  _buildNavItem(
                    activeIcon: Icons.favorite_rounded,
                    inactiveIcon: Icons.favorite_border_rounded,
                    label: 'Wishlist',
                    isSelected: widget.currentPage == 'wishlist',
                    badgeCount: wishlistCount,
                    onTap: () async {
                      if (widget.currentPage != 'wishlist') {
                        try {
                          context.pushNamed('WishlistProducts');
                        } catch (_) {}
                      }
                    },
                  ),

                  // 4. MY CART TAB
                  _buildNavItem(
                    activeIcon: Icons.shopping_cart_rounded,
                    inactiveIcon: Icons.shopping_cart_outlined,
                    label: 'My Cart',
                    isSelected: widget.currentPage == 'cart',
                    onTap: () async {
                      if (widget.currentPage != 'cart') {
                        try {
                          context.pushNamed('CartPage');
                        } catch (_) {}
                      }
                    },
                  ),

                  // 5. DYNAMIC PROFILE / SIGN IN TAB
                  if (!isGuest)
                    _buildNavItem(
                      activeIcon: Icons.person_rounded,
                      inactiveIcon: Icons.person_outline_rounded,
                      label: 'Profile',
                      isSelected: widget.currentPage == 'profile',
                      onTap: () async {
                        if (widget.currentPage != 'profile') {
                          try {
                            context.pushNamed('CustomerProfile');
                          } catch (_) {}
                        }
                      },
                    )
                  else
                    _buildNavItem(
                      activeIcon: Icons.login_rounded,
                      inactiveIcon: Icons.login_outlined,
                      label: 'Sign In',
                      isSelected: widget.currentPage == 'login',
                      onTap: () async {
                        try {
                          FFAppState().isGuest = false;
                          context.pushNamed('MainLoginPage');
                        } catch (_) {}
                      },
                    ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }
}

```

---

## Step 4: Place `DynamicLowerBar` on Your Pages

1. Disable FlutterFlow's default **Show Nav Bar** toggle on every page where you use `DynamicLowerBar`.
2. Structure each page's main `Column` with your scrollable content inside an **`Expanded`** widget at the top, and place **`DynamicLowerBar`** pinned at the bottom (`Width`: `Infinity` / `100%`, `Height`: `80`).
3. Set the **`currentPage`** parameter on each screen so the active tab highlights properly:

| Page Route Name | `currentPage` Value |
| --- | --- |
| **`HomeDashboard`** | `'home'` |
| **`CategoryExplorer`** | `'categories'` |
| **`WishlistProducts`** | `'wishlist'` |
| **`CartPage`** | `'cart'` |
| **`CustomerProfile`** | `'profile'` |
| **`MainLoginPage`** | `'login'` |
| **Secondary / Detail Pages** | `''` *(leaves all 5 tabs unselected)* |

*(Tip: Always match route names in `context.goNamed(...)` and `context.pushNamed(...)` to the exact **Route Name** string shown in each page's properties panel in FlutterFlow—case sensitivity matters.)*
