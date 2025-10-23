# Frontend Architecture

## Metadata

- **Category**: Architecture
- **Last Update**: 2025-10-20
- **Owner**: CTO (Lucas Gaillard)
- **Status**: Active
- **Source Files**: `apps/web/src/app/**`, `apps/web/next.config.ts`

## Technical Overview

The frontend is built with **Next.js 15** using the App Router architecture, React 19, and TypeScript. The application follows a component-based architecture with Tailwind CSS + Shadcn/UI for styling and TanStack Query for server state management.

The routing structure uses route groups to separate public and protected areas, with Clerk middleware handling authentication at the edge. Components are organized into UI primitives (Shadcn), layout components, and feature-specific components.

## Architecture & Design

### App Router Structure

```
apps/web/src/app/
├── (protected)/              # Authenticated routes
│   └── dashboard/
│       ├── layout.tsx        # Dashboard layout
│       ├── page.tsx          # Main dashboard
│       ├── integrations/
│       │   └── github/setup/
│       └── repos/[repositoryId]/
│           ├── about/
│           ├── readme/
│           └── admin/
│
├── (public)/                 # Public routes
│   ├── (auth)/
│   │   ├── sign-in/[[...sign-in]]/page.tsx
│   │   └── sign-up/[[...sign-up]]/page.tsx
│   ├── layout.tsx
│   └── page.tsx              # Landing page
│
├── api/status/route.ts       # API route
├── layout.tsx                # Root layout
├── globals.css
└── middleware.ts             # Auth middleware
```

### Component Organization

```
apps/web/src/components/
├── ui/                       # Shadcn UI primitives (44 components)
│   ├── button.tsx
│   ├── input.tsx
│   ├── dialog.tsx
│   └── ...
├── layout/                   # Layout components
│   └── sidebar/
│       ├── app-sidebar.tsx
│       ├── nav-user.tsx
│       └── logout-alert-dialog.tsx
└── github/                   # Feature components
    ├── repositories-switcher.tsx
    ├── manage-repositories-modal.tsx
    └── connect-github.tsx
```

### Middleware Configuration

**File**: `apps/web/src/middleware.ts`

```typescript
import { clerkMiddleware, createRouteMatcher } from "@clerk/nextjs/server";

const isPrivateRoute = createRouteMatcher(["/dashboard(.*)"]);

export default clerkMiddleware(async (auth, request) => {
  if (isPrivateRoute(request)) {
    await auth.protect(); // Protect dashboard routes
  }
});

export const config = {
  matcher: [
    "/((?!_next|[^?]*\\.(?:html?|css|js(?!on)|jpe?g|webp|png|gif|svg|ttf|woff2?|ico|csv|docx?|xlsx?|zip|webmanifest)).*)",
    "/(api|trpc)(.*)",
  ],
};
```

## Configuration & Setup

### Next.js Configuration

**File**: `apps/web/next.config.ts`

```typescript
const nextConfig: NextConfig = {
  output: "standalone", // Optimized for containers
  images: {
    remotePatterns: [
      { hostname: "api.dicebear.com" },
      { hostname: "img.clerk.com" },
    ],
  },
  async rewrites() {
    return [
      // PostHog proxying
      {
        source: "/ingest/static/:path*",
        destination: "https://eu-assets.i.posthog.com/static/:path*",
      },
      {
        source: "/ingest/:path*",
        destination: "https://eu.i.posthog.com/:path*",
      },
    ];
  },
  skipTrailingSlashRedirect: true, // PostHog compatibility
};

export default withSentryConfig(withNextIntl(nextConfig), {
  org: "boucleai",
  project: "web",
  authToken: process.env.SENTRY_AUTH_TOKEN,
  // ... Sentry config
});
```

## Patterns & Best Practices

### ✅ DO: Use Functional Components

```typescript
type UserProfileProps = {
  id: string;
};

export function UserProfile(props: UserProfileProps) {
  const { id } = props;
  const { data } = useQuery(['user', id], () => getUser(id));

  return data ? <div>{data.name}</div> : null;
}
```

### ✅ DO: Use TanStack Query for Data Fetching

```typescript
// apps/web/src/lib/data/github/use-github-repositories.tsx
export function useGithubRepositories() {
  const client = useHttpClient();
  const queryClient = useQueryClient();

  const {
    data: repositories,
    isPending,
    error,
  } = useQuery({
    queryKey: [QUERY_KEYS.GITHUB_REPOSITORIES],
    queryFn: () =>
      client.get<GithubRepository[]>(API_PATHS.GITHUB.REPOSITORIES),
  });

  const addRepository = useMutation({
    mutationFn: (repositoryId: string) =>
      client.post(API_PATHS.GITHUB.REPOSITORIES, { body: { repositoryId } }),
    onSuccess: () => {
      queryClient.invalidateQueries({
        queryKey: [QUERY_KEYS.GITHUB_REPOSITORIES],
      });
    },
  });

  return {
    repositories,
    isPending,
    error,
    addRepository: addRepository.mutateAsync,
  };
}
```

### ✅ DO: Organize by Feature

```
components/
├── ui/           # Reusable UI primitives
├── layout/       # Layout components (sidebar, nav)
└── github/       # Feature-specific components
```

### ❌ DON'T: Use Classes

```typescript
// BAD
class Profile extends React.Component {
  render() { return <div>{this.props.name}</div> }
}

// GOOD
export function Profile(props: ProfileProps) {
  const { name } = props;
  return <div>{name}</div>;
}
```

## Usage Examples

### Page Component with Data Fetching

```typescript
// apps/web/src/app/(protected)/dashboard/page.tsx
'use client';

export default function DashboardPage() {
  const { repositories = [], isPending } = useGithubRepositories();
  const router = useRouter();

  useEffect(() => {
    if (repositories.length > 0) {
      router.push(formatDashboardRepositoryPagePath(repositories[0].id));
    }
  }, [repositories, router]);

  if (isPending) return <Loader centered />;
  if (repositories.length === 0) return <RepositoriesSwitcher />;

  return <div>{repositories.length} repositories</div>;
}
```

### Custom Hook Pattern

```typescript
export function useGithubRepositories() {
  const client = useHttpClient();
  const queryClient = useQueryClient();

  // Query
  const { data, isPending, error } = useQuery({...});

  // Mutations
  const addRepository = useMutation({...});
  const removeRepository = useMutation({...});

  // Return interface
  return {
    repositories: data,
    isPending,
    error,
    addRepository: addRepository.mutateAsync,
    removeRepository: removeRepository.mutateAsync,
  };
}
```

## Technical Dependencies

**Frontend Core**:

- `react`: ^19.0.0
- `react-dom`: ^19.0.0
- `next`: 15.3.5
- `typescript`: ^5.7.3

**State Management**:

- `@tanstack/react-query`: ^5.71.1
- `@tanstack/react-query-devtools`: ^5.71.1

**UI & Styling**:

- `tailwindcss`: ^4.1.11
- `@radix-ui/*`: Various versions (UI primitives)
- `lucide-react`: ^0.525.0 (icons)
- `class-variance-authority`: ^0.7.1 (variants)
- `tailwind-merge`: ^3.3.1 (class merging)

**Utilities**:

- `next-intl`: ^4.3.4 (i18n)
- `next-themes`: ^0.4.6 (dark mode)

## References

- **Main Files**: `apps/web/src/app/**`, `apps/web/next.config.ts`
- **Components**: `apps/web/src/components/**` (61 components)
- **Hooks**: `apps/web/src/lib/data/**`
- **External Docs**: [Next.js App Router](https://nextjs.org/docs/app), [TanStack Query](https://tanstack.com/query), [Shadcn/UI](https://ui.shadcn.com/)
