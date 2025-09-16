# DESIGN SYSTEM - AURA 2.0
## Visual Identity & Component Library

## 🎨 BRAND IDENTITY

### Color Palette

```scss
// Primary Colors
$primary-gradient: linear-gradient(135deg, #667EEA 0%, #764BA2 100%);
$primary-purple: #667EEA;
$primary-magenta: #764BA2;

// Secondary Colors
$secondary-pink: #F093FB;
$secondary-coral: #F5576C;
$secondary-orange: #FDA085;

// Accent Colors
$accent-teal: #4FACFE;
$accent-blue: #00F2FE;
$accent-green: #43E97B;
$accent-yellow: #FFA63D;

// Neutral Colors
$neutral-900: #111827; // Dark text
$neutral-800: #1F2937;
$neutral-700: #374151;
$neutral-600: #4B5563;
$neutral-500: #6B7280;
$neutral-400: #9CA3AF;
$neutral-300: #D1D5DB;
$neutral-200: #E5E7EB;
$neutral-100: #F3F4F6;
$neutral-50: #F9FAFB;
$white: #FFFFFF;

// Semantic Colors
$success: #10B981;
$warning: #F59E0B;
$error: #EF4444;
$info: #3B82F6;

// Mode-Specific Colors
$flash-color: #FDA085; // Orange - Energy
$alma-color: #764BA2; // Purple - Deep Connection
$aura-color: #667EEA; // Royal Blue - Premium
$radar-color: #43E97B; // Green - Discovery
$vibe-color: #F093FB; // Pink - Community

// Dark Mode
$dark-bg: #0F172A;
$dark-surface: #1E293B;
$dark-border: #334155;
```

### Typography

```scss
// Font Families
$font-display: 'Outfit', sans-serif; // Headers
$font-body: 'Inter', sans-serif; // Body text
$font-mono: 'JetBrains Mono', monospace; // Code

// Font Sizes
$text-xs: 0.75rem; // 12px
$text-sm: 0.875rem; // 14px
$text-base: 1rem; // 16px
$text-lg: 1.125rem; // 18px
$text-xl: 1.25rem; // 20px
$text-2xl: 1.5rem; // 24px
$text-3xl: 1.875rem; // 30px
$text-4xl: 2.25rem; // 36px
$text-5xl: 3rem; // 48px
$text-6xl: 3.75rem; // 60px

// Font Weights
$font-light: 300;
$font-regular: 400;
$font-medium: 500;
$font-semibold: 600;
$font-bold: 700;
$font-extrabold: 800;

// Line Heights
$leading-tight: 1.25;
$leading-normal: 1.5;
$leading-relaxed: 1.75;

// Letter Spacing
$tracking-tight: -0.025em;
$tracking-normal: 0;
$tracking-wide: 0.025em;
```

### Spacing System

```scss
// Spacing Scale (rem)
$space-0: 0; // 0px
$space-1: 0.25rem; // 4px
$space-2: 0.5rem; // 8px
$space-3: 0.75rem; // 12px
$space-4: 1rem; // 16px
$space-5: 1.25rem; // 20px
$space-6: 1.5rem; // 24px
$space-8: 2rem; // 32px
$space-10: 2.5rem; // 40px
$space-12: 3rem; // 48px
$space-16: 4rem; // 64px
$space-20: 5rem; // 80px
$space-24: 6rem; // 96px
$space-32: 8rem; // 128px
```

### Border Radius

```scss
$radius-none: 0;
$radius-sm: 0.125rem; // 2px
$radius-base: 0.25rem; // 4px
$radius-md: 0.375rem; // 6px
$radius-lg: 0.5rem; // 8px
$radius-xl: 0.75rem; // 12px
$radius-2xl: 1rem; // 16px
$radius-3xl: 1.5rem; // 24px
$radius-full: 9999px;
```

### Shadows

```scss
$shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.05);
$shadow-base: 0 1px 3px 0 rgb(0 0 0 / 0.1), 0 1px 2px -1px rgb(0 0 0 / 0.1);
$shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
$shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);
$shadow-xl: 0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1);
$shadow-2xl: 0 25px 50px -12px rgb(0 0 0 / 0.25);
$shadow-inner: inset 0 2px 4px 0 rgb(0 0 0 / 0.05);

// Colored shadows for cards
$shadow-purple: 0 20px 40px -15px rgba(102, 126, 234, 0.3);
$shadow-pink: 0 20px 40px -15px rgba(240, 147, 251, 0.3);
```

## 🧩 COMPONENTS

### Button Component

```tsx
// components/Button/Button.tsx
import { ButtonHTMLAttributes, forwardRef } from 'react';
import { cva, type VariantProps } from 'class-variance-authority';
import { cn } from '@/utils/cn';

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-lg font-medium transition-all focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50',
  {
    variants: {
      variant: {
        primary: 'bg-gradient-to-r from-primary-purple to-primary-magenta text-white hover:shadow-lg',
        secondary: 'bg-white text-neutral-900 border-2 border-neutral-200 hover:bg-neutral-50',
        ghost: 'hover:bg-neutral-100 hover:text-neutral-900',
        danger: 'bg-error text-white hover:bg-red-600',
        success: 'bg-success text-white hover:bg-green-600',
      },
      size: {
        sm: 'h-9 px-3 text-sm',
        md: 'h-11 px-4 text-base',
        lg: 'h-13 px-6 text-lg',
        xl: 'h-15 px-8 text-xl',
        icon: 'h-10 w-10',
      },
      fullWidth: {
        true: 'w-full',
      },
    },
    defaultVariants: {
      variant: 'primary',
      size: 'md',
    },
  }
);

export interface ButtonProps
  extends ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  isLoading?: boolean;
  leftIcon?: React.ReactNode;
  rightIcon?: React.ReactNode;
}

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  (
    {
      className,
      variant,
      size,
      fullWidth,
      isLoading,
      leftIcon,
      rightIcon,
      children,
      disabled,
      ...props
    },
    ref
  ) => {
    return (
      <button
        ref={ref}
        className={cn(buttonVariants({ variant, size, fullWidth }), className)}
        disabled={disabled || isLoading}
        {...props}
      >
        {isLoading ? (
          <svg
            className="mr-2 h-4 w-4 animate-spin"
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
          >
            <circle
              className="opacity-25"
              cx="12"
              cy="12"
              r="10"
              stroke="currentColor"
              strokeWidth="4"
            />
            <path
              className="opacity-75"
              fill="currentColor"
              d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
            />
          </svg>
        ) : (
          leftIcon
        )}
        {children}
        {!isLoading && rightIcon}
      </button>
    );
  }
);

Button.displayName = 'Button';
```

### Card Component

```tsx
// components/Card/ProfileCard.tsx
import { motion } from 'framer-motion';
import { Heart, X, Star, MapPin, CheckCircle } from 'lucide-react';
import { User } from '@/types';

interface ProfileCardProps {
  user: User;
  onLike: () => void;
  onPass: () => void;
  onSuperLike?: () => void;
}

export const ProfileCard = ({ user, onLike, onPass, onSuperLike }: ProfileCardProps) => {
  const [imageIndex, setImageIndex] = useState(0);

  return (
    <motion.div
      className="relative h-[600px] w-full max-w-[400px] overflow-hidden rounded-2xl bg-white shadow-2xl"
      whileHover={{ scale: 1.02 }}
      transition={{ type: 'spring', stiffness: 300 }}
    >
      {/* Image Carousel */}
      <div className="relative h-[65%] w-full">
        <img
          src={user.photos[imageIndex].url}
          alt={user.displayName}
          className="h-full w-full object-cover"
        />
        
        {/* Image Dots */}
        <div className="absolute top-4 left-0 right-0 flex justify-center gap-1">
          {user.photos.map((_, index) => (
            <button
              key={index}
              onClick={() => setImageIndex(index)}
              className={cn(
                'h-1 w-8 rounded-full transition-all',
                index === imageIndex ? 'bg-white' : 'bg-white/50'
              )}
            />
          ))}
        </div>
        
        {/* Gradient Overlay */}
        <div className="absolute inset-0 bg-gradient-to-t from-black/60 via-transparent to-transparent" />
        
        {/* User Info Overlay */}
        <div className="absolute bottom-0 left-0 right-0 p-6 text-white">
          <div className="flex items-center gap-2">
            <h3 className="text-2xl font-bold">{user.displayName}, {user.age}</h3>
            {user.verificationLevel !== 'NONE' && (
              <CheckCircle className="h-6 w-6 text-blue-400" />
            )}
          </div>
          
          <div className="mt-2 flex items-center gap-4 text-sm">
            <span className="flex items-center gap-1">
              <MapPin className="h-4 w-4" />
              {user.distance}km away
            </span>
            <span className="rounded-full bg-white/20 px-3 py-1 backdrop-blur">
              {user.pronouns}
            </span>
          </div>
        </div>
      </div>
      
      {/* Bio Section */}
      <div className="p-6">
        <p className="mb-4 text-neutral-700 line-clamp-3">{user.bio}</p>
        
        {/* Interests */}
        <div className="flex flex-wrap gap-2">
          {user.interests.slice(0, 5).map((interest) => (
            <span
              key={interest}
              className="rounded-full bg-neutral-100 px-3 py-1 text-sm text-neutral-700"
            >
              {interest}
            </span>
          ))}
        </div>
      </div>
      
      {/* Action Buttons */}
      <div className="absolute bottom-0 left-0 right-0 flex justify-center gap-4 bg-gradient-to-t from-white via-white to-transparent p-6">
        <motion.button
          whileTap={{ scale: 0.9 }}
          onClick={onPass}
          className="flex h-14 w-14 items-center justify-center rounded-full bg-white shadow-lg hover:shadow-xl"
        >
          <X className="h-6 w-6 text-red-500" />
        </motion.button>
        
        {onSuperLike && (
          <motion.button
            whileTap={{ scale: 0.9 }}
            onClick={onSuperLike}
            className="flex h-14 w-14 items-center justify-center rounded-full bg-white shadow-lg hover:shadow-xl"
          >
            <Star className="h-6 w-6 text-blue-500" />
          </motion.button>
        )}
        
        <motion.button
          whileTap={{ scale: 0.9 }}
          onClick={onLike}
          className="flex h-14 w-14 items-center justify-center rounded-full bg-white shadow-lg hover:shadow-xl"
        >
          <Heart className="h-6 w-6 text-green-500" />
        </motion.button>
      </div>
    </motion.div>
  );
};
```

### Input Component

```tsx
// components/Input/Input.tsx
import { InputHTMLAttributes, forwardRef } from 'react';
import { cn } from '@/utils/cn';

export interface InputProps extends InputHTMLAttributes<HTMLInputElement> {
  label?: string;
  error?: string;
  hint?: string;
  leftIcon?: React.ReactNode;
  rightIcon?: React.ReactNode;
}

export const Input = forwardRef<HTMLInputElement, InputProps>(
  ({ className, label, error, hint, leftIcon, rightIcon, ...props }, ref) => {
    return (
      <div className="w-full">
        {label && (
          <label className="mb-2 block text-sm font-medium text-neutral-700">
            {label}
          </label>
        )}
        
        <div className="relative">
          {leftIcon && (
            <div className="pointer-events-none absolute inset-y-0 left-0 flex items-center pl-3">
              {leftIcon}
            </div>
          )}
          
          <input
            ref={ref}
            className={cn(
              'w-full rounded-lg border bg-white px-4 py-3 text-neutral-900 placeholder-neutral-400 transition-all',
              'focus:border-primary-purple focus:outline-none focus:ring-2 focus:ring-primary-purple/20',
              'disabled:bg-neutral-50 disabled:text-neutral-500',
              error && 'border-error focus:border-error focus:ring-error/20',
              leftIcon && 'pl-10',
              rightIcon && 'pr-10',
              className
            )}
            {...props}
          />
          
          {rightIcon && (
            <div className="absolute inset-y-0 right-0 flex items-center pr-3">
              {rightIcon}
            </div>
          )}
        </div>
        
        {hint && !error && (
          <p className="mt-1 text-sm text-neutral-500">{hint}</p>
        )}
        
        {error && (
          <p className="mt-1 text-sm text-error">{error}</p>
        )}
      </div>
    );
  }
);

Input.displayName = 'Input';
```

### Modal Component

```tsx
// components/Modal/MatchModal.tsx
import { motion, AnimatePresence } from 'framer-motion';
import { Heart, MessageCircle, Share2 } from 'lucide-react';
import confetti from 'canvas-confetti';

interface MatchModalProps {
  isOpen: boolean;
  onClose: () => void;
  matchedUser: User;
  onSendMessage: () => void;
}

export const MatchModal = ({
  isOpen,
  onClose,
  matchedUser,
  onSendMessage,
}: MatchModalProps) => {
  useEffect(() => {
    if (isOpen) {
      // Trigger confetti animation
      confetti({
        particleCount: 100,
        spread: 70,
        origin: { y: 0.6 },
        colors: ['#667EEA', '#764BA2', '#F093FB', '#F5576C'],
      });
    }
  }, [isOpen]);

  return (
    <AnimatePresence>
      {isOpen && (
        <motion.div
          initial={{ opacity: 0 }}
          animate={{ opacity: 1 }}
          exit={{ opacity: 0 }}
          className="fixed inset-0 z-50 flex items-center justify-center bg-black/60 backdrop-blur-sm"
          onClick={onClose}
        >
          <motion.div
            initial={{ scale: 0.8, opacity: 0 }}
            animate={{ scale: 1, opacity: 1 }}
            exit={{ scale: 0.8, opacity: 0 }}
            transition={{ type: 'spring', stiffness: 300 }}
            className="relative mx-4 w-full max-w-md overflow-hidden rounded-3xl bg-white shadow-2xl"
            onClick={(e) => e.stopPropagation()}
          >
            {/* Gradient Header */}
            <div className="relative h-48 bg-gradient-to-br from-primary-purple to-primary-magenta">
              <div className="absolute inset-0 flex items-center justify-center">
                <motion.div
                  initial={{ scale: 0 }}
                  animate={{ scale: 1 }}
                  transition={{ delay: 0.2 }}
                  className="rounded-full bg-white p-6"
                >
                  <Heart className="h-16 w-16 text-primary-magenta" fill="currentColor" />
                </motion.div>
              </div>
            </div>
            
            {/* Content */}
            <div className="p-8 text-center">
              <motion.h2
                initial={{ y: 20, opacity: 0 }}
                animate={{ y: 0, opacity: 1 }}
                transition={{ delay: 0.3 }}
                className="mb-2 text-3xl font-bold text-neutral-900"
              >
                It's a Match! 🎉
              </motion.h2>
              
              <motion.p
                initial={{ y: 20, opacity: 0 }}
                animate={{ y: 0, opacity: 1 }}
                transition={{ delay: 0.4 }}
                className="mb-6 text-neutral-600"
              >
                You and {matchedUser.displayName} liked each other!
              </motion.p>
              
              {/* Matched User Info */}
              <motion.div
                initial={{ y: 20, opacity: 0 }}
                animate={{ y: 0, opacity: 1 }}
                transition={{ delay: 0.5 }}
                className="mb-8 flex items-center justify-center gap-4"
              >
                <img
                  src={matchedUser.photos[0].url}
                  alt={matchedUser.displayName}
                  className="h-20 w-20 rounded-full object-cover ring-4 ring-white"
                />
                <div className="text-left">
                  <p className="font-semibold text-neutral-900">
                    {matchedUser.displayName}, {matchedUser.age}
                  </p>
                  <p className="text-sm text-neutral-500">
                    {matchedUser.distance}km away
                  </p>
                </div>
              </motion.div>
              
              {/* Actions */}
              <motion.div
                initial={{ y: 20, opacity: 0 }}
                animate={{ y: 0, opacity: 1 }}
                transition={{ delay: 0.6 }}
                className="flex gap-3"
              >
                <Button
                  variant="primary"
                  fullWidth
                  leftIcon={<MessageCircle className="h-5 w-5" />}
                  onClick={onSendMessage}
                >
                  Send a Message
                </Button>
                
                <Button
                  variant="secondary"
                  size="icon"
                  onClick={() => {
                    // Share functionality
                  }}
                >
                  <Share2 className="h-5 w-5" />
                </Button>
              </motion.div>
            </div>
          </motion.div>
        </motion.div>
      )}
    </AnimatePresence>
  );
};
```

## 📱 MOBILE DESIGN GUIDELINES

### iOS Design
```swift
// Colors.swift
struct AuraColors {
    static let primaryGradient = LinearGradient(
        colors: [Color(hex: "667EEA"), Color(hex: "764BA2")],
        startPoint: .topLeading,
        endPoint: .bottomTrailing
    )
    
    static let flashOrange = Color(hex: "FDA085")
    static let almaPurple = Color(hex: "764BA2")
    static let auraBlue = Color(hex: "667EEA")
    static let radarGreen = Color(hex: "43E97B")
    static let vibePink = Color(hex: "F093FB")
}

// Typography.swift
struct AuraTypography {
    static let largeTitle = Font.custom("Outfit-Bold", size: 34)
    static let title1 = Font.custom("Outfit-SemiBold", size: 28)
    static let title2 = Font.custom("Outfit-SemiBold", size: 22)
    static let title3 = Font.custom("Outfit-Medium", size: 20)
    static let headline = Font.custom("Inter-SemiBold", size: 17)
    static let body = Font.custom("Inter-Regular", size: 17)
    static let callout = Font.custom("Inter-Regular", size: 16)
    static let subheadline = Font.custom("Inter-Regular", size: 15)
    static let footnote = Font.custom("Inter-Regular", size: 13)
    static let caption1 = Font.custom("Inter-Regular", size: 12)
    static let caption2 = Font.custom("Inter-Regular", size: 11)
}
```

### Android Design
```kotlin
// Theme.kt
@Composable
fun AuraTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit
) {
    val colors = if (darkTheme) {
        darkColorScheme(
            primary = Purple80,
            secondary = PurpleGrey80,
            tertiary = Pink80
        )
    } else {
        lightColorScheme(
            primary = Purple40,
            secondary = PurpleGrey40,
            tertiary = Pink40
        )
    }

    MaterialTheme(
        colorScheme = colors,
        typography = AuraTypography,
        content = content
    )
}

// Typography.kt
val AuraTypography = Typography(
    displayLarge = TextStyle(
        fontFamily = FontFamily(Font(R.font.outfit_bold)),
        fontSize = 57.sp,
        lineHeight = 64.sp
    ),
    displayMedium = TextStyle(
        fontFamily = FontFamily(Font(R.font.outfit_semibold)),
        fontSize = 45.sp,
        lineHeight = 52.sp
    ),
    bodyLarge = TextStyle(
        fontFamily = FontFamily(Font(R.font.inter_regular)),
        fontSize = 16.sp,
        lineHeight = 24.sp
    )
)
```

## 🎭 ANIMATIONS

### Micro-interactions

```tsx
// animations/interactions.ts
export const springConfig = {
  type: 'spring',
  stiffness: 300,
  damping: 20,
};

export const likeAnimation = {
  initial: { scale: 0, rotate: -45 },
  animate: { scale: 1, rotate: 0 },
  exit: { scale: 0, rotate: 45, opacity: 0 },
  transition: springConfig,
};

export const matchCelebration = {
  initial: { scale: 0.5, opacity: 0 },
  animate: {
    scale: [0.5, 1.2, 1],
    opacity: 1,
    transition: {
      scale: {
        times: [0, 0.5, 1],
        duration: 0.6,
      },
    },
  },
};

export const cardSwipe = {
  drag: 'x',
  dragConstraints: { left: 0, right: 0 },
  dragElastic: 1,
  onDragEnd: (event, { offset, velocity }) => {
    const swipe = Math.abs(offset.x) * velocity.x;
    if (swipe > 10000) {
      // Trigger like/pass based on direction
      const direction = offset.x > 0 ? 'right' : 'left';
      handleSwipe(direction);
    }
  },
};
```

## 🌈 ACCESSIBILITY

### WCAG 2.1 AA Compliance

```scss
// Contrast ratios
// Normal text: 4.5:1 minimum
// Large text: 3:1 minimum
// UI components: 3:1 minimum

// Focus indicators
.focus-visible {
  outline: 2px solid $primary-purple;
  outline-offset: 2px;
}

// Screen reader only
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}

// Reduced motion
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

---

**Design Lead: UX/UI Department**
*Last Updated: January 2024*