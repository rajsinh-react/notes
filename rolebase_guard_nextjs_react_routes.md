🧩 1. Setup Example Roles

Assume roles like:

// Example roles
export const ROLES = {
  ADMIN: 'admin',
  USER: 'user',
  GUEST: 'guest',
};

🧱 2. Create Auth Context (or use Zustand / Redux)

Let’s use React Context for simplicity:

// context/AuthContext.js
"use client";
import { createContext, useContext, useState, useEffect } from "react";

const AuthContext = createContext();

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null); // { name: 'Raj', role: 'admin' }

  useEffect(() => {
    // Example: get user from localStorage or API
    const savedUser = JSON.parse(localStorage.getItem("user"));
    if (savedUser) setUser(savedUser);
  }, []);

  const login = (role) => {
    const u = { name: "Test User", role };
    setUser(u);
    localStorage.setItem("user", JSON.stringify(u));
  };

  const logout = () => {
    setUser(null);
    localStorage.removeItem("user");
  };

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => useContext(AuthContext);


Then wrap your app in this provider:

// app/layout.js
import { AuthProvider } from "@/context/AuthContext";

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>
        <AuthProvider>{children}</AuthProvider>
      </body>
    </html>
  );
}

🧰 3. Create Role-Based Route Guard

For protecting pages dynamically (works in Next.js App Router):

// components/RoleGuard.js
"use client";
import { useAuth } from "@/context/AuthContext";
import { useRouter } from "next/navigation";
import { useEffect } from "react";

export default function RoleGuard({ children, allowedRoles = [] }) {
  const { user } = useAuth();
  const router = useRouter();

  useEffect(() => {
    if (!user) {
      router.push("/login");
      return;
    }

    if (!allowedRoles.includes(user.role)) {
      router.push("/unauthorized");
    }
  }, [user, router, allowedRoles]);

  if (!user || !allowedRoles.includes(user.role)) return null;

  return <>{children}</>;
}

🧭 4. Use the Guard on Protected Pages

Example:

// app/admin/page.js
"use client";
import RoleGuard from "@/components/RoleGuard";
import { ROLES } from "@/constants/roles";

export default function AdminPage() {
  return (
    <RoleGuard allowedRoles={[ROLES.ADMIN]}>
      <h1>Admin Dashboard</h1>
    </RoleGuard>
  );
}

🧩 5. Role-Based Navigation Links

Only show links allowed for user’s role:

// components/NavBar.js
"use client";
import Link from "next/link";
import { useAuth } from "@/context/AuthContext";
import { ROLES } from "@/constants/roles";

export default function NavBar() {
  const { user, logout } = useAuth();

  const links = [
    { href: "/", label: "Home", roles: [ROLES.ADMIN, ROLES.USER, ROLES.GUEST] },
    { href: "/admin", label: "Admin", roles: [ROLES.ADMIN] },
    { href: "/profile", label: "Profile", roles: [ROLES.ADMIN, ROLES.USER] },
  ];

  return (
    <nav>
      {links
        .filter((l) => !user || l.roles.includes(user.role))
        .map((l) => (
          <Link key={l.href} href={l.href}>
            {l.label}
          </Link>
        ))}
      {user ? (
        <button onClick={logout}>Logout</button>
      ) : (
        <Link href="/login">Login</Link>
      )}
    </nav>
  );
}

⚙️ 6. Optional — Middleware Protection (Server Side)

If you want server-side route blocking, create:

// middleware.js
import { NextResponse } from "next/server";

export function middleware(req) {
  const url = req.nextUrl.clone();
  const role = req.cookies.get("role")?.value;

  if (url.pathname.startsWith("/admin") && role !== "admin") {
    url.pathname = "/unauthorized";
    return NextResponse.redirect(url);
  }

  return NextResponse.next();
}
