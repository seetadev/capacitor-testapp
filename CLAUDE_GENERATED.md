# Claude Generated Code

**Task**: Objective:
Help design and implement new features based on the current roadmap or project needs. Focus on clean, scalable, and testable code

Claude Tasks:

🧩 Feature Implementation
Read the existing codebase and integrate new feature modules where appropriate.

Ensure new code follows existing patterns and conventions.

🗂️ Example Features (replace/add specifics as needed):
Add user role-based access control

Implement export to PDF

Add multi-language support

Enable dark mode toggle

🧪 Testing & Documentation
Add unit/integration tests for new functionality.

Update README or in-code comments if required.

**Generated on**: Mon Jul 28 15:12:27 UTC 2025

## Generated Content

```
I'll implement a comprehensive government billing/invoicing feature for your Ionic React TypeScript application. This will include user management, invoice creation, PDF export, and role-based access control.

## FILES_TO_CREATE

1. `src/types/billing.ts`
2. `src/types/auth.ts`
3. `src/services/authService.ts`
4. `src/services/billingService.ts`
5. `src/services/pdfService.ts`
6. `src/hooks/useAuth.ts`
7. `src/hooks/useBilling.ts`
8. `src/components/AuthGuard.tsx`
9. `src/components/InvoiceForm.tsx`
10. `src/components/InvoiceList.tsx`
11. `src/components/UserRoleSelector.tsx`
12. `src/pages/Login.tsx`
13. `src/pages/Dashboard.tsx`
14. `src/pages/Invoices.tsx`
15. `src/pages/Users.tsx`
16. `src/context/AuthContext.tsx`
17. `src/utils/validation.ts`
18. `src/utils/formatting.ts`

## FILES_TO_MODIFY

1. `package.json`
2. `src/routes.tsx`
3. `src/App.tsx`
4. `src/components/Menu.tsx`

## CODE_CHANGES

### 1. package.json (Add dependencies)

```json
{
  "dependencies": {
    // ... existing dependencies
    "jspdf": "^2.5.1",
    "jspdf-autotable": "^3.5.31",
    "date-fns": "^2.30.0",
    "yup": "^1.3.3",
    "react-hook-form": "^7.47.0",
    "@hookform/resolvers": "^3.3.2"
  }
}
```

### 2. src/types/auth.ts

```typescript
export interface User {
  id: string;
  email: string;
  firstName: string;
  lastName: string;
  role: UserRole;
  department: string;
  isActive: boolean;
  createdAt: Date;
  updatedAt: Date;
}

export enum UserRole {
  ADMIN = 'admin',
  BILLING_MANAGER = 'billing_manager',
  BILLING_CLERK = 'billing_clerk',
  VIEWER = 'viewer'
}

export interface AuthState {
  user: User | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  error: string | null;
}

export interface LoginCredentials {
  email: string;
  password: string;
}

export interface RegisterData {
  email: string;
  password: string;
  firstName: string;
  lastName: string;
  department: string;
}
```

### 3. src/types/billing.ts

```typescript
export interface Invoice {
  id: string;
  invoiceNumber: string;
  clientName: string;
  clientAddress: string;
  clientEmail: string;
  items: InvoiceItem[];
  subtotal: number;
  taxRate: number;
  taxAmount: number;
  total: number;
  status: InvoiceStatus;
  issueDate: Date;
  dueDate: Date;
  createdBy: string;
  createdAt: Date;
  updatedAt: Date;
  notes?: string;
}

export interface InvoiceItem {
  id: string;
  description: string;
  quantity: number;
  unitPrice: number;
  total: number;
}

export enum InvoiceStatus {
  DRAFT = 'draft',
  SENT = 'sent',
  PAID = 'paid',
  OVERDUE = 'overdue',
  CANCELLED = 'cancelled'
}

export interface InvoiceFilters {
  status?: InvoiceStatus;
  startDate?: Date;
  endDate?: Date;
  clientName?: string;
}

export interface BillingStats {
  totalInvoices: number;
  totalRevenue: number;
  pendingAmount: number;
  overdueAmount: number;
}
```

### 4. src/context/AuthContext.tsx

```typescript
import React, { createContext, useContext, useReducer, useEffect } from 'react';
import { AuthState, User, LoginCredentials, RegisterData } from '../types/auth';
import { authService } from '../services/authService';

interface AuthContextType extends AuthState {
  login: (credentials: LoginCredentials) => Promise<void>;
  register: (data: RegisterData) => Promise<void>;
  logout: () => void;
  clearError: () => void;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

type AuthAction =
  | { type: 'AUTH_START' }
  | { type: 'AUTH_SUCCESS'; payload: User }
  | { type: 'AUTH_ERROR'; payload: string }
  | { type: 'AUTH_LOGOUT' }
  | { type: 'CLEAR_ERROR' };

const authReducer = (state: AuthState, action: AuthAction): AuthState => {
  switch (action.type) {
    case 'AUTH_START':
      return { ...state, isLoading: true, error: null };
    case 'AUTH_SUCCESS':
      return {
        ...state,
        user: action.payload,
        isAuthenticated: true,
        isLoading: false,
        error: null,
      };
    case 'AUTH_ERROR':
      return {
        ...state,
        isLoading: false,
        error: action.payload,
        isAuthenticated: false,
        user: null,
      };
    case 'AUTH_LOGOUT':
      return {
        ...state,
        user: null,
        isAuthenticated: false,
        isLoading: false,
        error: null,
      };
    case 'CLEAR_ERROR':
      return { ...state, error: null };
    default:
      return state;
  }
};

const initialState: AuthState = {
  user: null,
  isAuthenticated: false,
  isLoading: false,
  error: null,
};

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [state, dispatch] = useReducer(authReducer, initialState);

  useEffect(() => {
    const initAuth = async () => {
      dispatch({ type: 'AUTH_START' });
      try {
        const user = await authService.getCurrentUser();
        if (user) {
          dispatch({ type: 'AUTH_SUCCESS', payload: user });
        } else {
          dispatch({ type: 'AUTH_LOGOUT' });
        }
      } catch (error) {
        dispatch({ type: 'AUTH_LOGOUT' });
      }
    };

    initAuth();
  }, []);

  const login = async (credentials: LoginCredentials) => {
    dispatch({ type: 'AUTH_START' });
    try {
      const user = await authService.login(credentials);
      dispatch({ type: 'AUTH_SUCCESS', payload: user });
    } catch (error) {
      dispatch({ type: 'AUTH_ERROR', payload: error instanceof Error ? error.message : 'Login failed' });
    }
  };

  const register = async (data: RegisterData) => {
    dispatch({ type: 'AUTH_START' });
    try {
      const user = await authService.register(data);
      dispatch({ type: 'AUTH_SUCCESS', payload: user });
    } catch (error) {
      dispatch({ type: 'AUTH_ERROR', payload: error instanceof Error ? error.message : 'Registration failed' });
    }
  };

  const logout = () => {
    authService.logout();
    dispatch({ type: 'AUTH_LOGOUT' });
  };

  const clearError = () => {
    dispatch({ type: 'CLEAR_ERROR' });
  };

  const value: AuthContextType = {
    ...state,
    login,
    register,
    logout,
    clearError,
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
};

export const useAuth = (): AuthContextType => {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};
```

### 5. src/services/authService.ts

```typescript
import { Preferences } from '@capacitor/preferences';
import { User, LoginCredentials, RegisterData, UserRole } from '../types/auth';

const AUTH_TOKEN_KEY = 'auth_token';
const USER_DATA_KEY = 'user_data';

class AuthService {
  private mockUsers: User[] = [
    {
      id: '1',
      email: 'admin@gov.local',
      firstName: 'Admin',
      lastName: 'User',
      role: UserRole.ADMIN,
      department: 'Administration',
      isActive: true,
      createdAt: new Date(),
      updatedAt: new Date(),
    },
    {
      id: '2',
      email: 'billing@gov.local',
      firstName: 'Billing',
      lastName: 'Manager',
      role: UserRole.BILLING_MANAGER,
      department: 'Finance',
      isActive: true,
      createdAt: new Date(),
      updatedAt: new Date(),
    },
  ];

  async login(credentials: LoginCredentials): Promise<User> {
    // Simulate API call
    await this.delay(1000);

    const user = this.mockUsers.find(u => u.email === credentials.email);
    if (!user || credentials.password !== 'password123') {
      throw new Error('Invalid credentials');
    }

    const token = this.generateToken();
    await Preferences.set({ key: AUTH_TOKEN_KEY, value: token });
    await Preferences.set({ key: USER_DATA_KEY, value: JSON.stringify(user) });

    return user;
  }

  async register(data: RegisterData): Promise<User> {
    // Simulate API call
    await this.delay(1000);

    // Check if user already exists
    const existingUser = this.mockUsers.find(u => u.email === data.email);
    if (existingUser) {
      throw new Error('User already exists');
    }

    const newUser: User = {
      id: Date.now().toString(),
      email: data.email,
      firstName: data.firstName,
      lastName: data.lastName,
      role: UserRole.BILLING_CLERK,
      department: data.department,
      isActive: true,
      createdAt: new Date(),
      updatedAt: new Date(),
    };

    this.mockUsers.push(newUser);

    const token = this.generateToken();
    await Preferences.set({ key: AUTH_TOKEN_KEY, value: token });
    await Preferences.set({ key: USER_DATA_KEY, value: JSON.stringify(newUser) });

    return newUser;
  }

  async getCurrentUser(): Promise<User | null> {
    try {
      const { value: token } = await Preferences.get({ key: AUTH_TOKEN_KEY });
      const { value: userData } = await Preferences.get({ key: USER_DATA_KEY });

      if (!token || !userData) {
        return null;
      }

      return JSON.parse(userData);
    } catch (error) {
      return null;
    }
  }

  async logout(): Promise<void> {
    await Preferences.remove({ key: AUTH_TOKEN_KEY });
    await Preferences.remove({ key: USER_DATA_KEY });
  }

  async getUsers(): Promise<User[]> {
    await this.delay(500);
    return this.mockUsers;
  }

  async updateUserRole(userId: string, role: UserRole): Promise<User> {
    await this.delay(500);
    
    const userIndex = this.mockUsers.findIndex(u => u.id === userId);
    if (userIndex === -1) {
      throw new Error('User not found');
    }

    this.mockUsers[userIndex] = {
      ...this.mockUsers[userIndex],
      role,
      updatedAt: new Date(),
    };

    return this.mockUsers[userIndex];
  }

  private generateToken(): string {
    return Math.random().toString(36).substr(2) + Date.now().toString(36);
  }

  private delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

export const authService = new AuthService();
```

### 6. src/services/billingService.ts

```typescript
import { Invoice, InvoiceItem, InvoiceStatus, InvoiceFilters, BillingStats } from '../types/billing';
import { format } from 'date-fns';

class BillingService {
  private mockInvoices: Invoice[] = [
    {
      id: '1',
      invoiceNumber: 'INV-2024-001',
      clientName: 'City Department of Transportation',
      clientAddress: '123 Main St, City, State 12345',
      clientEmail: 'billing@transport.gov',
      items: [
        {
          id: '1',
          description: 'Professional Services - Q1 2024',
          quantity: 1,
          unitPrice: 15000,
          total: 15000,
        },
      ],
      subtotal: 15000,
      taxRate: 8.5,
      taxAmount: 1275,
      total: 16275,
      status: InvoiceStatus.SENT,
      issueDate: new Date('2024-01-15'),
      dueDate: new Date('2024-02-15'),
      createdBy: '1',
      createdAt: new Date('2024-01-15'),
      updatedAt: new Date('2024-01-15'),
      notes: 'Payment terms: Net 30 days',
    },
  ];

  async getInvoices(filters?: InvoiceFilters): Promise<Invoice[]> {
    await this.delay(500);
    
    let filtered = [...this.mockInvoices];

    if (filters) {
      if (filters.status) {
        filtered = filtered.filter(inv => inv.status === filters.status);
      }
      if (filters.clientName) {
        filtered = filtered.filter(inv => 
          inv.clientName.toLowerCase().includes(filters.clientName!.toLowerCase())
        );
      }
      if (filters.startDate) {
        filtered = filtered.filter(inv => inv.issueDate >= filters.startDate!);
      }
      if (filters.endDate) {
        filtered = filtered.filter(inv => inv.issueDate <= filters.endDate!);
      }
    }

    return filtered;
  }

  async getInvoice(id: string): Promise<Invoice | null> {
    await this.delay(300);
    return this.mockInvoices.find(inv => inv.id === id) || null;
  }

  async createInvoice(invoiceData: Omit<Invoice, 'id' | 'invoiceNumber' | 'createdAt' | 'updatedAt'>): Promise<Invoice> {
    await this.delay(1000);

    const newInvoice: Invoice = {
      ...invoiceData,
      id: Date.now().toString(),
      invoiceNumber: this.generateInvoiceNumber(),
      createdAt: new Date(),
      updatedAt: new Date(),
    };

    this.mockInvoices.push(newInvoice);
    return newInvoice;
  }

  async updateInvoice(id: string, updates: Partial<Invoice>): Promise<Invoice> {
    await this.delay(500);

    const index = this.mockInvoices.findIndex(inv => inv.id === id);
    if (index === -1) {
      throw new Error('Invoice not found');
    }

    this.mockInvoices[index] = {
      ...this.mockInvoices[index],
      ...updates,
      updatedAt: new Date(),
    };

    return this.mockInvoices[index];
  }

  async deleteInvoice(id: string): Promise<void> {
    await this.delay(500);

    const index = this.mockInvoices.findIndex(inv => inv.id === id);
    if (index === -1) {
      throw new Error('Invoice not found');
    }

    this.mockInvoices.splice(index, 1);
  }

  async getBillingStats(): Promise<BillingStats> {
    await this.delay(500);

    const totalInvoices = this.mockInvoices.length;
    const totalRevenue = this.mockInvoices
      .filter(inv => inv.status === InvoiceStatus.PAID)
      .reduce((sum, inv) => sum + inv.total, 0);
    
    const pendingAmount = this.mockInvoices
      .filter(inv => inv.status === InvoiceStatus.SENT)
      .reduce((sum, inv) => sum + inv.total, 0);

    const overdueAmount = this.mockInvoices
      .filter(inv => inv.status === InvoiceStatus.OVERDUE)
      .reduce((sum, inv) => sum + inv.total, 0);

    return {
      totalInvoices,
      totalRevenue,
      pendingAmount,
      overdueAmount,
    };
  }

  private generateInvoiceNumber(): string {
    const year = new Date().getFullYear();
    const count = this.mockInvoices.length + 1;
    return `INV-${year}-${count.toString().padStart(3, '0')}`;
  }

  private delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

export const billingService = new BillingService();
```

### 7. src/services/pdfService.ts

```typescript
import jsPDF from 'jspdf';
import autoTable from 'jspdf-autotable';
import { Invoice } from '../types/billing';
import { format } from 'date-fns';

class PDFService {
  async generateInvoicePDF(invoice: Invoice): Promise<void> {
    const doc = new jsPDF();
    
    // Header
    doc.setFontSize(20);
    doc.text('GOVERNMENT INVOICE', 20, 30);
    
    // Invoice details
    doc.setFontSize(12);
    doc.text(`Invoice Number: ${invoice.invoiceNumber}`, 20, 50);
    doc.text(`Issue Date: ${format(invoice.issueDate, 'MM/dd/yyyy')}`, 20, 60);
    doc.text(`Due Date: ${format(invoice.dueDate, 'MM/dd/yyyy')}`, 20, 70);
    doc.text(`Status: ${invoice.status.toUpperCase()}`, 20, 80);
    
    // Client information
    doc.text('Bill To:', 120, 50);
    doc.text(invoice.clientName, 120, 60);
    doc.text(invoice.clientAddress, 120, 70);
    doc.text(invoice.clientEmail, 120, 80);
    
    // Items table
    const tableData = invoice.items.map(item => [
      item.description,
      item.quantity.toString(),
      `$${item.unitPrice.toFixed(2)}`,
      `$${item.total.toFixed(2)}`,
    ]);
    
    autoTable(doc, {
      startY: 100,
      head: [['Description', 'Quantity', 'Unit Price', 'Total']],
      body: tableData,
      theme: 'grid',
    });
    
    // Totals
    const finalY = (doc as any).lastAutoTable.finalY || 150;
    doc.text(`Subtotal: $${invoice.subtotal.toFixed(2)}`, 140, finalY + 20);
    doc.text(`Tax (${invoice.taxRate}%): $${invoice.taxAmount.toFixed(2)}`, 140, finalY + 30);
    doc.setFontSize(14);
    doc.text(`Total: $${invoice.total.toFixed(2)}`, 140, finalY + 45);
    
    // Notes
    if (invoice.notes) {
      doc.setFontSize(10);
      doc.text('Notes:', 20, finalY + 60);
      doc.text(invoice.notes, 20, finalY + 70);
    }
    
    // Save the PDF
    doc.save(`${invoice.invoiceNumber}.pdf`);
  }
}

export const pdfService = new PDFService();
```

### 8. src/components/AuthGuard.tsx

```typescript
import React from 'react';
import { useAuth } from '../context/AuthContext';
import { UserRole } from '../types/auth';
import { IonContent, IonText } from '@ionic/react';

interface AuthGuardProps {
  children: React.ReactNode;
  requiredRoles?: UserRole[];
  fallback?: React.ReactNode;
}

export const AuthGuard: React.FC<AuthGuardProps> = ({
  children,
  requiredRoles = [],
  fallback
}) => {
  const { user, isAuthenticated, isLoading } = useAuth();

  if (isLoading) {
    return (
      <IonContent>
        <div style={{ display: 'flex', justifyContent: 'center', alignItems: 'center', height: '100%' }}>
          <IonText>Loading...</IonText>
        </div>
      </IonContent>
    );
  }

  if (!isAuthenticated || !user) {
    return fallback || (
      <IonContent>
        <div style={{ display: 'flex', justifyContent: 'center', alignItems: 'center', height: '100%' }}>
          <IonText color="danger">Access Denied. Please log in.</IonText>
        </div>
      </IonContent>
    );
  }

  if (requiredRoles.length > 0 && !requiredRoles.includes(user.role)) {
    return fallback || (
      <IonContent>
        <div style={{ display: 'flex', justifyContent: 'center', alignItems: 'center', height: '100%' }}>
          <IonText color="danger">Insufficient permissions.</IonText>
        </div>
      </IonContent>
    );
  }

  return <>{children}</>;
};
```

### 9. src/pages/Login.tsx

```typescript
import React, { useState, useEffect } from 'react';
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
  IonItem,
  IonLabel,
  IonInput,
  IonButton,
  IonText,
  IonCard,
  IonCardContent,
  IonCardHeader,
  IonCardTitle,
  IonLoading,
} from '@ionic/react';
import { useAuth } from '../context/AuthContext';
import { useHistory } from 'react-router-dom';

const Login: React.FC = () => {
  const [email, setEmail] = useState('admin@gov.local');
  const [password, setPassword] = useState('password123');
  const { login, isLoading, error, isAuthenticated, clearError } = useAuth();
  const history = useHistory();

  useEffect(() => {
    if (isAuthenticated) {
      history.replace('/dashboard');
    }
  }, [isAuthenticated, history]);

  useEffect(() => {
    return () => {
      clearError();
    };
  }, [clearError]);

  const handleLogin = async () => {
    if (!email || !password) return;
    
    try {
      await login({ email, password });
    } catch (error) {
      // Error is handled by the auth context
    }
  };

  return (
    <IonPage>
      <IonHeader>
        <IonToolbar>
          <IonTitle>Government Billing System</IonTitle>
        </IonToolbar>
      </IonHeader>
      <IonContent className="ion-padding">
        <div style={{ display: 'flex', justifyContent: 'center', alignItems: 'center', minHeight: '60vh' }}>
          <IonCard style={{ width: '100%', maxWidth: '400px' }}>
            <IonCardHeader>
              <IonCardTitle>Sign In</IonCardTitle>
            </IonCardHeader>
            <IonCardContent>
              <IonItem>
                <IonLabel position="stacked">Email</IonLabel>
                <IonInput
                  type="email"
                  value={email}
                  onIonInput={(e) => setEmail(e.detail.value!)}
                  placeholder="Enter your email"
                />
              </IonItem>
              <IonItem>
                <IonLabel position="stacked">Password</IonLabel>
                <IonInput
                  type="password"
                  value={password}
                  onIonInput={(e) => setPassword(e.detail.value!)}
                  placeholder="Enter your password"
                />
              </IonItem>
              
              {error && (
                <IonText color="danger">
                  <p>{error}</p>
                </IonText>
              )}
              
              <IonButton
                expand="full"
                onClick={handleLogin}
                disabled={isLoading || !email || !password}
                style={{ marginTop: '20px' }}
              >
                Sign In
              </IonButton>
              
              <div style={{ marginTop: '20px', textAlign: 'center' }}>
                <IonText color="medium">
                  <p>Demo Credentials:</p>
                  <p>admin@gov.local / password123</p>
                  <p>billing@gov.local / password123</p>
                </IonText>
              </div>
            </IonCardContent>
          </IonCard>
        </div>
        <IonLoading isOpen={isLoading} message="Signing in..." />
      </IonContent>
    </IonPage>
  );
};

export default Login;
```

### 10. src/pages/Dashboard.tsx

```typescript
import React, { useEffect, useState } from 'react';
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
  IonCard,
  IonCardContent,
  IonCardHeader,
  IonCardTitle,
  IonGrid,
  IonRow,
  IonCol,
  IonText,
  IonButton,
  IonButtons,
  IonMenuButton,
  IonIcon,
} from '@ionic/react';
import { documentsOutline, cashOutline, timeOutline, alertCircleOutline } from 'ionicons/icons';
import { useAuth } from '../context/AuthContext';
import { billingService } from '../services/billingService';
import { BillingStats } from '../types/billing';
import { AuthGuard } from '../components/AuthGuard';

const Dashboard: React.FC = () => {
  const { user } = useAuth();
  const [stats, setStats] = useState<BillingStats | null>(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    const loadStats = async () => {
      try {
        const billingStats = await billingService.getBillingStats();
        setStats(billingStats);
      } catch (error) {
        console.error('Failed to load stats:', error);
      } finally {
        setIsLoading(false);
      }
    };

    loadStats();
  }, []);

  return (
    <IonPage>
      <IonHeader>
        <IonToolbar>
          <IonButtons slot="start">
            <IonMenuButton />
          </IonButtons>
          <IonTitle>Dashboard</IonTitle>
        </IonToolbar>
      </IonHeader>
      <IonContent className="ion-padding">
        <AuthGuard>
          <div style={{ marginBottom: '20px' }}>
            <IonText>
              <h2>Welcome, {user?.firstName} {user?.lastName}</h2>
              <p>Role: {user?.role} | Department: {user?.department}</p>
            </IonText>
          </div>

          {!isLoading && stats && (
            <IonGrid>
              <IonRow>
                <IonCol size="12" sizeMd="6" sizeLg="3">
                  <IonCard>
                    <IonCardContent>
                      <div style={{ display: 'flex', alignItems: 'center', justifyContent: 'space-between' }}>
                        <div>
                          <IonText color="medium">
                            <p>Total Invoices</p>
                          </IonText>
                          <IonText>
                            <h2>{stats.totalInvoices}</h2>
                          </IonText>
                        </div>
                        <IonIcon icon={documentsOutline} size="large" color="primary" />
                      </div>
                    </IonCardContent>
                  </IonCard>
                </IonCol>

                <IonCol size="12" sizeMd="6" sizeLg="3">
                  <IonCard>
                    <IonCardContent>
                      <div style={{ display: 'flex', alignItems: 'center', justifyContent: 'space-between' }}>
                        <div>
                          <IonText color="medium">
                            <p>Total Revenue</p>
                          </IonText>
                          <IonText>
                            <h2>${stats.totalRevenue.toLocaleString()}</h2>
                          </IonText>
                        </div>
                        <IonIcon icon={cashOutline} size="large" color="success" />
                      </div>
                    </IonCardContent>
                  </IonCard>
                </IonCol>

                <IonCol size="12" sizeMd="6" sizeLg="3">
                  <IonCard>
                    <IonCardContent>
                      <div style={{ display: 'flex', alignItems: 'center', justifyContent: 'space-between' }}>
                        <div>
                          <IonText color="medium">
                            <p>Pending Amount</p>
                          </IonText>
                          <IonText>
                            <h2>${stats.pendingAmount.toLocaleString()}</h2>
                          </IonText>
                        </div>
                        <IonIcon icon={timeOutline} size="large" color="warning" />
                      </div>
                    </IonCardContent>
                  </IonCard>
                </IonCol>

                <IonCol size="12" sizeMd="6" sizeLg="3">
                  <IonCard>
                    <IonCardContent>
                      <div style={{ display: 'flex', alignItems: 'center', justifyContent: 'space-between' }}>
                        <div>
                          <IonText color="medium">
                            <p>Overdue Amount</p>
                          </IonText>
                          <IonText>
                            <h2>${stats.overdueAmount.toLocaleString()}
```
