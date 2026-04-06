# App Service - Danh sach function

- Pham vi quet: app_service/backend/app + app_service/src
- Thoi diem tao: 2026-04-06 14:19:43
- So file co function: 86
- Tong so function tim thay: 386

## app_service/backend/app/api/auth_routes.py

- _user_public (line 39, python-def)
- login (line 45, python-def)
- register (line 71, python-def)
- bootstrap_first_admin (line 101, python-def)
- read_me (line 126, python-def)
- recover_password (line 136, python-def)
- change_password (line 158, python-def)

## app_service/backend/app/api/authorizations_routes.py

- list_authorizations (line 23, python-def)
- create_authorization (line 52, python-def)

## app_service/backend/app/api/devices_routes.py

- list_devices_admin (line 36, python-def)
- create_device (line 46, python-def)
- patch_device (line 72, python-def)
- list_devices_for_current_user (line 93, python-def)
- delete_device (line 111, python-def)
- get_device (line 128, python-def)

## app_service/backend/app/api/health.py

- health (line 17, python-def)
- health_db (line 23, python-def)

## app_service/backend/app/api/mqtt_routes.py

- _get_mqtt (line 15, python-def)
- mqtt_status (line 24, python-def)
- mqtt_messages (line 31, python-def)

## app_service/backend/app/api/users_routes.py

- list_users (line 28, python-def)
- patch_user_status (line 52, python-def)
- delete_user (line 70, python-def)

## app_service/backend/app/core/config.py

- settings_customise_sources (line 32, python-def)
- database_url (line 74, python-def)

## app_service/backend/app/core/db.py

- _pymysql_connect (line 20, python-def)
- get_engine (line 32, python-def)
- db_ping (line 47, python-def)

## app_service/backend/app/core/db_migrate.py

- ensure_user_expired_at_column (line 18, python-def)
- ensure_device_user_device_asignment_id_column (line 48, python-def)
- ensure_device_authorization_granted_by_varchar (line 74, python-def)
- ensure_device_drop_last_reading_columns (line 105, python-def)
- ensure_device_ui_columns (line 126, python-def)

## app_service/backend/app/core/db_wait.py

- wait_for_db (line 17, python-def)
- _ping (line 30, python-def)

## app_service/backend/app/core/deps.py

- get_db (line 25, python-def)
- get_current_user (line 38, python-def)
- require_admin (line 73, python-def)

## app_service/backend/app/core/mqtt_subscriber.py

- _parse_topics (line 38, python-def)
- **init** (line 50, python-def)
- start (line 89, python-def)
- stop (line 104, python-def)
- status (line 117, python-def)
- message_count (line 132, python-def)
- latest_messages (line 137, python-def)
- _on_connect (line 153, python-def)
- _on_disconnect (line 165, python-def)
- _on_message (line 171, python-def)

## app_service/backend/app/core/security.py

- hash_password (line 21, python-def)
- verify_password (line 26, python-def)
- create_access_token (line 34, python-def)
- decode_token (line 54, python-def)

## app_service/backend/app/core/seed.py

- ensure_default_admin (line 26, python-def)
- ensure_default_devices (line 53, python-def)

## app_service/backend/app/core/user_expiry.py

- deactivate_expired_users (line 14, python-def)

## app_service/backend/app/main.py

- lifespan (line 40, python-def)
- create_app (line 87, python-def)

## app_service/backend/app/schemas/auth.py

- cccd_digits (line 37, python-def)
- expired_not_in_past (line 44, python-def)
- validity_days (line 67, python-def)
- remaining_days (line 75, python-def)
- cccd_digits (line 98, python-def)
- cccd_digits (line 131, python-def)

## app_service/src/App.jsx

- App (line 8, function-declaration)

## app_service/src/components/AddDeviceModal.jsx

- AddDeviceModal (line 5, arrow-function)
- validateForm (line 26, arrow-function)
- handleChange (line 47, arrow-function)
- handleSubmit (line 54, arrow-function)

## app_service/src/components/AdminRoute.jsx

- AdminRoute (line 10, arrow-function)

## app_service/src/components/AssignDeviceModal.jsx

- todayIso (line 5, function-declaration)
- plusDaysIso (line 9, function-declaration)
- AssignDeviceModal (line 15, function-declaration)
- load (line 29, arrow-function)
- handleSubmit (line 69, arrow-function)

## app_service/src/components/ChangePasswordModal.jsx

- ChangePasswordModal (line 4, arrow-function)
- validateForm (line 13, arrow-function)
- handleChange (line 40, arrow-function)
- handleSubmit (line 47, arrow-function)

## app_service/src/components/IoTApp.jsx

- IoTApp (line 27, function-declaration)

## app_service/src/components/Layout.jsx

- Layout (line 12, arrow-function)
- isNavItemActive (line 27, arrow-function)

## app_service/src/components/ProtectedRoute.jsx

- ProtectedRoute (line 10, arrow-function)

## app_service/src/components/ui/accordion.tsx

- Accordion (line 7, function-declaration)
- AccordionItem (line 13, function-declaration)
- AccordionTrigger (line 26, function-declaration)
- AccordionContent (line 48, function-declaration)

## app_service/src/components/ui/alert.tsx

- Alert (line 22, function-declaration)
- AlertTitle (line 37, function-declaration)
- AlertDescription (line 50, function-declaration)

## app_service/src/components/ui/alert-dialog.tsx

- AlertDialog (line 9, function-declaration)
- AlertDialogTrigger (line 15, function-declaration)
- AlertDialogPortal (line 23, function-declaration)
- AlertDialogOverlay (line 31, function-declaration)
- AlertDialogContent (line 47, function-declaration)
- AlertDialogHeader (line 66, function-declaration)
- AlertDialogFooter (line 79, function-declaration)
- AlertDialogTitle (line 95, function-declaration)
- AlertDialogDescription (line 108, function-declaration)
- AlertDialogAction (line 121, function-declaration)
- AlertDialogCancel (line 133, function-declaration)

## app_service/src/components/ui/aspect-ratio.tsx

- AspectRatio (line 3, function-declaration)

## app_service/src/components/ui/avatar.tsx

- Avatar (line 8, function-declaration)
- AvatarImage (line 24, function-declaration)
- AvatarFallback (line 37, function-declaration)

## app_service/src/components/ui/badge.tsx

- Badge (line 28, function-declaration)

## app_service/src/components/ui/breadcrumb.tsx

- Breadcrumb (line 7, function-declaration)
- BreadcrumbList (line 11, function-declaration)
- BreadcrumbItem (line 24, function-declaration)
- BreadcrumbLink (line 34, function-declaration)
- BreadcrumbPage (line 52, function-declaration)
- BreadcrumbSeparator (line 65, function-declaration)
- BreadcrumbEllipsis (line 83, function-declaration)

## app_service/src/components/ui/button.tsx

- Button (line 38, function-declaration)

## app_service/src/components/ui/calendar.tsx

- Calendar (line 12, function-declaration)
- CalendarDayButton (line 170, function-declaration)

## app_service/src/components/ui/card.tsx

- Card (line 5, function-declaration)
- CardHeader (line 18, function-declaration)
- CardTitle (line 31, function-declaration)
- CardDescription (line 41, function-declaration)
- CardAction (line 51, function-declaration)
- CardContent (line 64, function-declaration)
- CardFooter (line 74, function-declaration)

## app_service/src/components/ui/carousel.tsx

- useCarousel (line 35, function-declaration)
- Carousel (line 45, function-declaration)
- CarouselContent (line 135, function-declaration)
- CarouselItem (line 156, function-declaration)
- CarouselPrevious (line 174, function-declaration)
- CarouselNext (line 204, function-declaration)

## app_service/src/components/ui/chart.tsx

- useChart (line 25, function-declaration)
- ChartContainer (line 35, function-declaration)
- ChartStyle (line 70, arrow-function)
- ChartTooltipContent (line 105, function-declaration)
- ChartLegendContent (line 251, function-declaration)
- getPayloadConfigFromPayload (line 306, function-declaration)

## app_service/src/components/ui/checkbox.tsx

- Checkbox (line 9, function-declaration)

## app_service/src/components/ui/collapsible.tsx

- Collapsible (line 3, function-declaration)
- CollapsibleTrigger (line 9, function-declaration)
- CollapsibleContent (line 20, function-declaration)

## app_service/src/components/ui/command.tsx

- Command (line 16, function-declaration)
- CommandDialog (line 32, function-declaration)
- CommandInput (line 63, function-declaration)
- CommandList (line 85, function-declaration)
- CommandEmpty (line 101, function-declaration)
- CommandGroup (line 113, function-declaration)
- CommandSeparator (line 129, function-declaration)
- CommandItem (line 142, function-declaration)
- CommandShortcut (line 158, function-declaration)

## app_service/src/components/ui/context-menu.tsx

- ContextMenu (line 9, function-declaration)
- ContextMenuTrigger (line 15, function-declaration)
- ContextMenuGroup (line 23, function-declaration)
- ContextMenuPortal (line 31, function-declaration)
- ContextMenuSub (line 39, function-declaration)
- ContextMenuRadioGroup (line 45, function-declaration)
- ContextMenuSubTrigger (line 56, function-declaration)
- ContextMenuSubContent (line 80, function-declaration)
- ContextMenuContent (line 96, function-declaration)
- ContextMenuItem (line 114, function-declaration)
- ContextMenuCheckboxItem (line 137, function-declaration)
- ContextMenuRadioItem (line 163, function-declaration)
- ContextMenuLabel (line 187, function-declaration)
- ContextMenuSeparator (line 207, function-declaration)
- ContextMenuShortcut (line 220, function-declaration)

## app_service/src/components/ui/dialog.tsx

- Dialog (line 7, function-declaration)
- DialogTrigger (line 13, function-declaration)
- DialogPortal (line 19, function-declaration)
- DialogClose (line 25, function-declaration)
- DialogOverlay (line 31, function-declaration)
- DialogContent (line 47, function-declaration)
- DialogHeader (line 81, function-declaration)
- DialogFooter (line 91, function-declaration)
- DialogTitle (line 104, function-declaration)
- DialogDescription (line 117, function-declaration)

## app_service/src/components/ui/drawer.tsx

- Drawer (line 6, function-declaration)
- DrawerTrigger (line 12, function-declaration)
- DrawerPortal (line 18, function-declaration)
- DrawerClose (line 24, function-declaration)
- DrawerOverlay (line 30, function-declaration)
- DrawerContent (line 46, function-declaration)
- DrawerHeader (line 73, function-declaration)
- DrawerFooter (line 86, function-declaration)
- DrawerTitle (line 96, function-declaration)
- DrawerDescription (line 109, function-declaration)

## app_service/src/components/ui/dropdown-menu.tsx

- DropdownMenu (line 9, function-declaration)
- DropdownMenuPortal (line 15, function-declaration)
- DropdownMenuTrigger (line 23, function-declaration)
- DropdownMenuContent (line 34, function-declaration)
- DropdownMenuGroup (line 54, function-declaration)
- DropdownMenuItem (line 62, function-declaration)
- DropdownMenuCheckboxItem (line 85, function-declaration)
- DropdownMenuRadioGroup (line 111, function-declaration)
- DropdownMenuRadioItem (line 122, function-declaration)
- DropdownMenuLabel (line 146, function-declaration)
- DropdownMenuSeparator (line 166, function-declaration)
- DropdownMenuShortcut (line 179, function-declaration)
- DropdownMenuSub (line 195, function-declaration)
- DropdownMenuSubTrigger (line 201, function-declaration)
- DropdownMenuSubContent (line 225, function-declaration)

## app_service/src/components/ui/form.tsx

- useFormField (line 43, arrow-function)
- FormItem (line 74, function-declaration)
- FormLabel (line 88, function-declaration)
- FormControl (line 105, function-declaration)
- FormDescription (line 123, function-declaration)
- FormMessage (line 136, function-declaration)

## app_service/src/components/ui/hover-card.tsx

- HoverCard (line 6, function-declaration)
- HoverCardTrigger (line 12, function-declaration)
- HoverCardContent (line 20, function-declaration)

## app_service/src/components/ui/input.tsx

- Input (line 5, function-declaration)

## app_service/src/components/ui/input-otp.tsx

- InputOTP (line 9, function-declaration)
- InputOTPGroup (line 29, function-declaration)
- InputOTPSlot (line 39, function-declaration)
- InputOTPSeparator (line 69, function-declaration)

## app_service/src/components/ui/label.tsx

- Label (line 8, function-declaration)

## app_service/src/components/ui/menubar.tsx

- Menubar (line 7, function-declaration)
- MenubarMenu (line 23, function-declaration)
- MenubarGroup (line 29, function-declaration)
- MenubarPortal (line 35, function-declaration)
- MenubarRadioGroup (line 41, function-declaration)
- MenubarTrigger (line 49, function-declaration)
- MenubarContent (line 65, function-declaration)
- MenubarItem (line 89, function-declaration)
- MenubarCheckboxItem (line 112, function-declaration)
- MenubarRadioItem (line 138, function-declaration)
- MenubarLabel (line 162, function-declaration)
- MenubarSeparator (line 182, function-declaration)
- MenubarShortcut (line 195, function-declaration)
- MenubarSub (line 211, function-declaration)
- MenubarSubTrigger (line 217, function-declaration)
- MenubarSubContent (line 241, function-declaration)

## app_service/src/components/ui/navigation-menu.tsx

- NavigationMenu (line 8, function-declaration)
- NavigationMenuList (line 32, function-declaration)
- NavigationMenuItem (line 48, function-declaration)
- NavigationMenuTrigger (line 65, function-declaration)
- NavigationMenuContent (line 85, function-declaration)
- NavigationMenuViewport (line 102, function-declaration)
- NavigationMenuLink (line 124, function-declaration)
- NavigationMenuIndicator (line 140, function-declaration)

## app_service/src/components/ui/pagination.tsx

- Pagination (line 11, function-declaration)
- PaginationContent (line 23, function-declaration)
- PaginationItem (line 36, function-declaration)
- PaginationLink (line 45, function-declaration)
- PaginationPrevious (line 68, function-declaration)
- PaginationNext (line 85, function-declaration)
- PaginationEllipsis (line 102, function-declaration)

## app_service/src/components/ui/popover.tsx

- Popover (line 8, function-declaration)
- PopoverTrigger (line 14, function-declaration)
- PopoverContent (line 20, function-declaration)
- PopoverAnchor (line 42, function-declaration)

## app_service/src/components/ui/progress.tsx

- Progress (line 6, function-declaration)

## app_service/src/components/ui/radio-group.tsx

- RadioGroup (line 9, function-declaration)
- RadioGroupItem (line 22, function-declaration)

## app_service/src/components/ui/resizable.tsx

- ResizablePanelGroup (line 7, function-declaration)
- ResizablePanel (line 23, function-declaration)
- ResizableHandle (line 29, function-declaration)

## app_service/src/components/ui/scroll-area.tsx

- ScrollArea (line 8, function-declaration)
- ScrollBar (line 31, function-declaration)

## app_service/src/components/ui/select.tsx

- Select (line 7, function-declaration)
- SelectGroup (line 13, function-declaration)
- SelectValue (line 19, function-declaration)
- SelectTrigger (line 25, function-declaration)
- SelectContent (line 51, function-declaration)
- SelectLabel (line 86, function-declaration)
- SelectItem (line 99, function-declaration)
- SelectSeparator (line 123, function-declaration)
- SelectScrollUpButton (line 136, function-declaration)
- SelectScrollDownButton (line 154, function-declaration)

## app_service/src/components/ui/separator.tsx

- Separator (line 8, function-declaration)

## app_service/src/components/ui/sheet.tsx

- Sheet (line 7, function-declaration)
- SheetTrigger (line 11, function-declaration)
- SheetClose (line 17, function-declaration)
- SheetPortal (line 23, function-declaration)
- SheetOverlay (line 29, function-declaration)
- SheetContent (line 45, function-declaration)
- SheetHeader (line 82, function-declaration)
- SheetFooter (line 92, function-declaration)
- SheetTitle (line 102, function-declaration)
- SheetDescription (line 115, function-declaration)

## app_service/src/components/ui/sidebar.tsx

- useSidebar (line 47, function-declaration)
- SidebarProvider (line 56, function-declaration)
- handleKeyDown (line 98, arrow-function)
- Sidebar (line 154, function-declaration)
- SidebarTrigger (line 256, function-declaration)
- SidebarRail (line 282, function-declaration)
- SidebarInset (line 307, function-declaration)
- SidebarInput (line 321, function-declaration)
- SidebarHeader (line 335, function-declaration)
- SidebarFooter (line 346, function-declaration)
- SidebarSeparator (line 357, function-declaration)
- SidebarContent (line 371, function-declaration)
- SidebarGroup (line 385, function-declaration)
- SidebarGroupLabel (line 396, function-declaration)
- SidebarGroupAction (line 417, function-declaration)
- SidebarGroupContent (line 440, function-declaration)
- SidebarMenu (line 454, function-declaration)
- SidebarMenuItem (line 465, function-declaration)
- SidebarMenuButton (line 498, function-declaration)
- SidebarMenuAction (line 548, function-declaration)
- SidebarMenuBadge (line 580, function-declaration)
- SidebarMenuSkeleton (line 602, function-declaration)
- SidebarMenuSub (line 640, function-declaration)
- SidebarMenuSubItem (line 655, function-declaration)
- SidebarMenuSubButton (line 669, function-declaration)

## app_service/src/components/ui/skeleton.tsx

- Skeleton (line 3, function-declaration)

## app_service/src/components/ui/slider.tsx

- Slider (line 8, function-declaration)

## app_service/src/components/ui/sonner.tsx

- Toaster (line 4, arrow-function)

## app_service/src/components/ui/switch.tsx

- Switch (line 8, function-declaration)

## app_service/src/components/ui/table.tsx

- Table (line 5, function-declaration)
- TableHeader (line 20, function-declaration)
- TableBody (line 30, function-declaration)
- TableFooter (line 40, function-declaration)
- TableRow (line 53, function-declaration)
- TableHead (line 66, function-declaration)
- TableCell (line 79, function-declaration)
- TableCaption (line 92, function-declaration)

## app_service/src/components/ui/tabs.tsx

- Tabs (line 8, function-declaration)
- TabsList (line 21, function-declaration)
- TabsTrigger (line 37, function-declaration)
- TabsContent (line 53, function-declaration)

## app_service/src/components/ui/textarea.tsx

- Textarea (line 5, function-declaration)

## app_service/src/components/ui/toggle.tsx

- Toggle (line 29, function-declaration)

## app_service/src/components/ui/toggle-group.tsx

- ToggleGroup (line 17, function-declaration)
- ToggleGroupItem (line 43, function-declaration)

## app_service/src/components/ui/tooltip.tsx

- TooltipProvider (line 6, function-declaration)
- Tooltip (line 19, function-declaration)
- TooltipTrigger (line 29, function-declaration)
- TooltipContent (line 35, function-declaration)

## app_service/src/contexts/AuthContext.jsx

- useAuth (line 16, arrow-function)
- AuthProvider (line 24, arrow-function)
- login (line 53, arrow-function)
- logout (line 66, arrow-function)
- isAdmin (line 73, arrow-function)

## app_service/src/data/mockData.js

- generateTemperatureData (line 3, arrow-function)
- generateHumidityData (line 18, arrow-function)
- generateDeviceHistory (line 102, arrow-function)
- mockRecentAlerts (line 201, arrow-function)
- pick (line 203, arrow-function)

## app_service/src/hooks/use-mobile.ts

- useIsMobile (line 5, function-declaration)
- onChange (line 10, arrow-function)

## app_service/src/lib/api.js

- apiFetch (line 14, function-declaration)

## app_service/src/lib/utils.ts

- cn (line 4, function-declaration)

## app_service/src/pages/ChangePassword.jsx

- ChangePassword (line 6, function-declaration)
- validate (line 16, arrow-function)
- handleSubmit (line 26, arrow-function)

## app_service/src/pages/Dashboard.jsx

- Dashboard (line 6, arrow-function)
- StatCard (line 10, arrow-function)
- DeviceStatusCard (line 25, arrow-function)

## app_service/src/pages/DeviceDetail.jsx

- mapApiDeviceToUi (line 39, function-declaration)
- isoDateToDisplay (line 56, function-declaration)
- formatTime (line 64, function-declaration)
- capPush (line 72, function-declaration)
- DeviceDetail (line 77, arrow-function)
- load (line 99, arrow-function)
- connect (line 145, arrow-function)
- maskPassword (line 221, arrow-function)

## app_service/src/pages/Devices.jsx

- Devices (line 9, arrow-function)
- load (line 24, arrow-function)
- handleToggleStatus (line 115, arrow-function)
- handleDeviceClick (line 123, arrow-function)
- handleAddDevice (line 127, arrow-function)
- handleDeleteDevice (line 133, arrow-function)
- DeviceCard (line 186, arrow-function)

## app_service/src/pages/Forbidden.jsx

- Forbidden (line 5, function-declaration)

## app_service/src/pages/ForgotPassword.jsx

- ForgotPassword (line 6, arrow-function)
- validateForm (line 13, arrow-function)
- handleSubmit (line 22, arrow-function)

## app_service/src/pages/GlobalDashboard.jsx

- formatTime (line 19, function-declaration)
- capPush (line 27, function-declaration)
- GlobalDashboard (line 32, function-declaration)
- connect (line 54, arrow-function)

## app_service/src/pages/Home.jsx

- Home (line 5, function-declaration)
- StatCard (line 8, arrow-function)
- statusLabel (line 23, arrow-function)

## app_service/src/pages/Login.jsx

- Login (line 6, arrow-function)
- validateForm (line 14, arrow-function)
- handleSubmit (line 35, arrow-function)

## app_service/src/pages/UserManagement.jsx

- defaultExpiredAt (line 7, function-declaration)
- isoToDdMmYyyy (line 14, function-declaration)
- ddMmYyyyToIso (line 22, function-declaration)
- todayIso (line 37, function-declaration)
- emptyForm (line 41, arrow-function)
- UserManagement (line 53, function-declaration)
- openModal (line 144, arrow-function)
- handleStatusChange (line 153, arrow-function)
- handleRegister (line 169, arrow-function)
- openDelete (line 220, arrow-function)
- openAssign (line 226, arrow-function)
- handleDelete (line 230, arrow-function)