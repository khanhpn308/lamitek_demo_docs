# App Service — Danh sách function

> Tài liệu **tự sinh** bởi `app_service/scripts/gen_functions_doc.py`.
> Là bản đồ index tra cứu nhanh: hàm/component nằm ở file nào, dòng nào.
> KHÔNG sửa tay — chạy lại script để cập nhật.

- Phạm vi quét: `app_service/backend/app` + `app_service/src`
- Thời điểm tạo: 2026-07-17 18:15:15
- Số file có function: 107
- Tổng số function tìm thấy: 508
## app_service/backend/app/api/auth_routes.py

- _user_public (dòng 40, python-def)
- login (dòng 47, python-def)
- register (dòng 73, python-def)
- bootstrap_first_admin (dòng 104, python-def)
- read_me (dòng 129, python-def)
- recover_password (dòng 140, python-def)
- change_password (dòng 162, python-def)

## app_service/backend/app/api/authorizations_routes.py

- list_authorizations (dòng 23, python-def)
- create_authorization (dòng 52, python-def)

## app_service/backend/app/api/devices_routes.py

- _sync_topic_runtime (dòng 38, python-def)
- list_devices_admin (dòng 50, python-def)
- create_device (dòng 68, python-def)
- patch_device (dòng 97, python-def)
- list_device_topics (dòng 127, python-def)
- update_device_topic (dòng 146, python-def)
- list_devices_for_current_user (dòng 182, python-def)
- delete_device (dòng 200, python-def)
- get_device (dòng 217, python-def)

## app_service/backend/app/api/floorplans_routes.py

- get_floorplan_dir (dòng 12, python-def)
- get_floorplan_webp (dòng 37, python-def)

## app_service/backend/app/api/health.py

- health (dòng 17, python-def)
- health_db (dòng 23, python-def)

## app_service/backend/app/api/locations_routes.py

- get_floorplan_dir (dòng 8, python-def)
- get_locations (dòng 37, python-def)

## app_service/backend/app/api/mqtt_routes.py

- _get_mqtt (dòng 26, python-def)
- _get_influx (dòng 34, python-def)
- mqtt_status (dòng 43, python-def)
- mqtt_messages (dòng 50, python-def)
- mqtt_topics (dòng 61, python-def)
- mqtt_subscribe_topic (dòng 71, python-def)
- mqtt_unsubscribe_topic (dòng 83, python-def)
- mqtt_history (dòng 95, python-def)
- mqtt_influx_status (dòng 108, python-def)

## app_service/backend/app/api/users_routes.py

- list_users (dòng 28, python-def)
- patch_user_status (dòng 63, python-def)
- delete_user (dòng 81, python-def)

## app_service/backend/app/api/websocket_routes.py

- _get_realtime_hub (dòng 43, python-def)
- ws_global (dòng 50, python-async-def)
- ws_device (dòng 99, python-async-def)
- ws_esp32 (dòng 190, python-async-def)

## app_service/backend/app/core/config.py

- settings_customise_sources (dòng 42, python-def)
- _guard_prod_secrets (dòng 93, python-def)
- database_url (dòng 117, python-def)

## app_service/backend/app/core/db.py

- _pymysql_connect (dòng 20, python-def)
- get_engine (dòng 32, python-def)
- db_ping (dòng 47, python-def)

## app_service/backend/app/core/db_migrate.py

- ensure_user_expired_at_column (dòng 18, python-def)
- ensure_device_user_device_asignment_id_column (dòng 48, python-def)
- ensure_device_authorization_granted_by_varchar (dòng 74, python-def)
- ensure_device_drop_last_reading_columns (dòng 105, python-def)
- ensure_device_ui_columns (dòng 126, python-def)
- ensure_device_topic_column (dòng 155, python-def)
- ensure_device_publish_topic_column (dòng 174, python-def)
- _column_exists (dòng 193, python-def)
- ensure_user_cccd_varchar (dòng 204, python-def)
- ensure_schema_hardening (dòng 243, python-def)

## app_service/backend/app/core/db_wait.py

- wait_for_db (dòng 17, python-async-def)
- _ping (dòng 30, python-def)

## app_service/backend/app/core/deps.py

- get_db (dòng 26, python-def)
- get_current_user (dòng 45, python-def)
- require_admin (dòng 80, python-def)
- _ws_extract_token (dòng 94, python-def)
- authenticate_ws_user (dòng 109, python-async-def)
- _load (dòng 131, python-def)
- authenticate_ws_device (dòng 142, python-async-def)
- _load (dòng 155, python-def)

## app_service/backend/app/core/influx_service.py

- _pick_metric (dòng 21, python-def)
- __init__ (dòng 39, python-def)
- start (dòng 62, python-def)
- stop (dòng 78, python-def)
- status (dòng 92, python-def)
- write_sensor_point (dòng 104, python-def)
- query_history (dòng 196, python-def)

## app_service/backend/app/core/ingest.py

- ingest_sensor_payload (dòng 24, python-def)

## app_service/backend/app/core/mqtt_subscriber.py

- _parse_topics (dòng 40, python-def)
- __init__ (dòng 52, python-def)
- start (dòng 95, python-def)
- stop (dòng 110, python-def)
- status (dòng 123, python-def)
- list_topics (dòng 138, python-def)
- subscribe_topic (dòng 148, python-def)
- unsubscribe_topic (dòng 172, python-def)
- message_count (dòng 197, python-def)
- latest_messages (dòng 202, python-def)
- publish_binary (dòng 218, python-def)
- _on_connect (dòng 239, python-def)
- _on_disconnect (dòng 253, python-def)
- _on_message (dòng 259, python-def)

## app_service/backend/app/core/payload_decoder.py

- _is_reasonable_epoch_seconds (dòng 41, python-def)
- _parse_iso_ts_to_epoch_utc (dòng 48, python-def)
- _extract_device_id_from_topic (dòng 63, python-def)
- _normalize_ts (dòng 77, python-def)
- _first_non_none (dòng 116, python-def)
- _normalize_sensor_type (dòng 127, python-def)
- _sensor_name_from_code (dòng 159, python-def)
- _read_varint (dòng 170, python-def)
- _skip_protobuf_value (dòng 188, python-def)
- _decode_simple_sensor_proto (dòng 214, python-def)
- _decode_nanopb_template (dòng 283, python-def)
- decode_sensor_payload (dòng 343, python-def)

## app_service/backend/app/core/realtime_hub.py

- __init__ (dòng 23, python-def)
- start (dòng 33, python-async-def)
- stop (dòng 41, python-async-def)
- connect_global (dòng 67, python-async-def)
- connect_device (dòng 73, python-async-def)
- connect_esp32 (dòng 80, python-async-def)
- disconnect_global (dòng 87, python-async-def)
- disconnect_device (dòng 92, python-async-def)
- disconnect_esp32 (dòng 103, python-async-def)
- send_to_esp32 (dòng 114, python-async-def)
- publish_from_thread (dòng 140, python-def)
- _push (dòng 150, python-def)
- _broadcast_worker (dòng 157, python-async-def)

## app_service/backend/app/core/security.py

- hash_password (dòng 21, python-def)
- verify_password (dòng 26, python-def)
- create_access_token (dòng 34, python-def)
- decode_token (dòng 54, python-def)

## app_service/backend/app/core/seed.py

- ensure_default_admin (dòng 25, python-def)
- ensure_default_devices (dòng 52, python-def)

## app_service/backend/app/core/test_payload_codec.py

- _read_u8 (dòng 11, python-def)
- _read_bytes (dòng 17, python-def)
- _read_len_ascii (dòng 23, python-def)
- _read_varint (dòng 29, python-def)
- _skip_protobuf_value (dòng 44, python-def)
- decode_coordinates_data_proto (dòng 67, python-def)
- decode_test_uplink_binary (dòng 158, python-def)
- _encode_varint (dòng 201, python-def)
- _encode_key (dòng 213, python-def)
- _encode_len_field (dòng 217, python-def)
- _encode_u64_field (dòng 222, python-def)
- encode_test_downlink_proto (dòng 226, python-def)

## app_service/backend/app/core/user_expiry.py

- deactivate_expired_users (dòng 14, python-def)

## app_service/backend/app/core/ws_manager.py

- __init__ (dòng 8, python-def)
- connect_device (dòng 14, python-async-def)
- disconnect_device (dòng 20, python-def)
- broadcast_to_dashboards (dòng 26, python-async-def)
- send_command_to_device (dòng 31, python-async-def)

## app_service/backend/app/main.py

- lifespan (dòng 55, python-async-def)
- _handle_sensor_payload (dòng 111, python-def)
- _resolve_ping_reply_topic (dòng 115, python-def)
- create_app (dòng 168, python-def)

## app_service/backend/app/schemas/auth.py

- cccd_digits (dòng 35, python-def)
- expired_not_in_past (dòng 42, python-def)
- validity_days (dòng 65, python-def)
- remaining_days (dòng 73, python-def)
- cccd_digits (dòng 96, python-def)
- cccd_digits (dòng 129, python-def)

## app_service/src/App.jsx

- App (dòng 8, function-declaration)

## app_service/src/components/AddDeviceModal.jsx

- AddDeviceModal (dòng 5, arrow-function)
- validateForm (dòng 25, arrow-function)
- handleChange (dòng 46, arrow-function)
- handleSubmit (dòng 53, arrow-function)

## app_service/src/components/AdminRoute.jsx

- AdminRoute (dòng 10, arrow-function)

## app_service/src/components/AssignDeviceModal.jsx

- todayIso (dòng 5, function-declaration)
- plusDaysIso (dòng 9, function-declaration)
- AssignDeviceModal (dòng 15, function-declaration)
- load (dòng 29, arrow-function)
- handleSubmit (dòng 69, arrow-function)

## app_service/src/components/ChangePasswordModal.jsx

- ChangePasswordModal (dòng 20, arrow-function)
- validateForm (dòng 29, arrow-function)
- handleChange (dòng 56, arrow-function)
- handleSubmit (dòng 63, arrow-function)
- handleOpenChange (dòng 75, arrow-function)
- inputClass (dòng 79, arrow-function)

## app_service/src/components/common/PageHeader.jsx

- PageHeader (dòng 9, function-declaration)

## app_service/src/components/common/Panel.jsx

- Panel (dòng 11, function-declaration)

## app_service/src/components/common/StatCard.jsx

- StatCard (dòng 15, function-declaration)

## app_service/src/components/common/StatusBadge.jsx

- StatusBadge (dòng 12, function-declaration)

## app_service/src/components/Dashboard/GPS/GPSDashboard.jsx

- getColor (dòng 5, arrow-function)
- GPSDashboard (dòng 15, arrow-function)
- fetchLocations (dòng 25, arrow-function)

## app_service/src/components/Dashboard/GPS/MapViewer.jsx

- MapViewer (dòng 3, arrow-function)

## app_service/src/components/devices/DeviceTable.tsx

- DeviceTable (dòng 12, arrow-function)

## app_service/src/components/devices/DeviceTableRow.tsx

- DeviceTableRow (dòng 10, arrow-function)

## app_service/src/components/devices/DeviceTableSkeleton.tsx

- DeviceTableSkeleton (dòng 4, arrow-function)

## app_service/src/components/IoTApp.jsx

- IoTApp (dòng 29, function-declaration)

## app_service/src/components/Layout.jsx

- Layout (dòng 12, arrow-function)
- isNavItemActive (dòng 29, arrow-function)

## app_service/src/components/ProtectedRoute.jsx

- ProtectedRoute (dòng 10, arrow-function)

## app_service/src/components/ui/accordion.tsx

- Accordion (dòng 7, function-declaration)
- AccordionItem (dòng 13, function-declaration)
- AccordionTrigger (dòng 26, function-declaration)
- AccordionContent (dòng 48, function-declaration)

## app_service/src/components/ui/alert-dialog.tsx

- AlertDialog (dòng 9, function-declaration)
- AlertDialogTrigger (dòng 15, function-declaration)
- AlertDialogPortal (dòng 23, function-declaration)
- AlertDialogOverlay (dòng 31, function-declaration)
- AlertDialogContent (dòng 47, function-declaration)
- AlertDialogHeader (dòng 66, function-declaration)
- AlertDialogFooter (dòng 79, function-declaration)
- AlertDialogTitle (dòng 95, function-declaration)
- AlertDialogDescription (dòng 108, function-declaration)
- AlertDialogAction (dòng 121, function-declaration)
- AlertDialogCancel (dòng 133, function-declaration)

## app_service/src/components/ui/alert.tsx

- Alert (dòng 22, function-declaration)
- AlertTitle (dòng 37, function-declaration)
- AlertDescription (dòng 50, function-declaration)

## app_service/src/components/ui/aspect-ratio.tsx

- AspectRatio (dòng 3, function-declaration)

## app_service/src/components/ui/avatar.tsx

- Avatar (dòng 8, function-declaration)
- AvatarImage (dòng 24, function-declaration)
- AvatarFallback (dòng 37, function-declaration)

## app_service/src/components/ui/badge.tsx

- Badge (dòng 28, function-declaration)

## app_service/src/components/ui/breadcrumb.tsx

- Breadcrumb (dòng 7, function-declaration)
- BreadcrumbList (dòng 11, function-declaration)
- BreadcrumbItem (dòng 24, function-declaration)
- BreadcrumbLink (dòng 34, function-declaration)
- BreadcrumbPage (dòng 52, function-declaration)
- BreadcrumbSeparator (dòng 65, function-declaration)
- BreadcrumbEllipsis (dòng 83, function-declaration)

## app_service/src/components/ui/button.tsx

- Button (dòng 38, function-declaration)

## app_service/src/components/ui/calendar.tsx

- Calendar (dòng 12, function-declaration)
- CalendarDayButton (dòng 170, function-declaration)

## app_service/src/components/ui/card.tsx

- Card (dòng 5, function-declaration)
- CardHeader (dòng 18, function-declaration)
- CardTitle (dòng 31, function-declaration)
- CardDescription (dòng 41, function-declaration)
- CardAction (dòng 51, function-declaration)
- CardContent (dòng 64, function-declaration)
- CardFooter (dòng 74, function-declaration)

## app_service/src/components/ui/carousel.tsx

- useCarousel (dòng 35, function-declaration)
- Carousel (dòng 45, function-declaration)
- CarouselContent (dòng 135, function-declaration)
- CarouselItem (dòng 156, function-declaration)
- CarouselPrevious (dòng 174, function-declaration)
- CarouselNext (dòng 204, function-declaration)

## app_service/src/components/ui/chart.tsx

- useChart (dòng 25, function-declaration)
- ChartContainer (dòng 35, function-declaration)
- ChartStyle (dòng 70, arrow-function)
- ChartTooltipContent (dòng 105, function-declaration)
- ChartLegendContent (dòng 251, function-declaration)
- getPayloadConfigFromPayload (dòng 306, function-declaration)

## app_service/src/components/ui/checkbox.tsx

- Checkbox (dòng 9, function-declaration)

## app_service/src/components/ui/collapsible.tsx

- Collapsible (dòng 3, function-declaration)
- CollapsibleTrigger (dòng 9, function-declaration)
- CollapsibleContent (dòng 20, function-declaration)

## app_service/src/components/ui/command.tsx

- Command (dòng 16, function-declaration)
- CommandDialog (dòng 32, function-declaration)
- CommandInput (dòng 63, function-declaration)
- CommandList (dòng 85, function-declaration)
- CommandEmpty (dòng 101, function-declaration)
- CommandGroup (dòng 113, function-declaration)
- CommandSeparator (dòng 129, function-declaration)
- CommandItem (dòng 142, function-declaration)
- CommandShortcut (dòng 158, function-declaration)

## app_service/src/components/ui/context-menu.tsx

- ContextMenu (dòng 9, function-declaration)
- ContextMenuTrigger (dòng 15, function-declaration)
- ContextMenuGroup (dòng 23, function-declaration)
- ContextMenuPortal (dòng 31, function-declaration)
- ContextMenuSub (dòng 39, function-declaration)
- ContextMenuRadioGroup (dòng 45, function-declaration)
- ContextMenuSubTrigger (dòng 56, function-declaration)
- ContextMenuSubContent (dòng 80, function-declaration)
- ContextMenuContent (dòng 96, function-declaration)
- ContextMenuItem (dòng 114, function-declaration)
- ContextMenuCheckboxItem (dòng 137, function-declaration)
- ContextMenuRadioItem (dòng 163, function-declaration)
- ContextMenuLabel (dòng 187, function-declaration)
- ContextMenuSeparator (dòng 207, function-declaration)
- ContextMenuShortcut (dòng 220, function-declaration)

## app_service/src/components/ui/dialog.tsx

- Dialog (dòng 7, function-declaration)
- DialogTrigger (dòng 13, function-declaration)
- DialogPortal (dòng 19, function-declaration)
- DialogClose (dòng 25, function-declaration)
- DialogOverlay (dòng 31, function-declaration)
- DialogContent (dòng 47, function-declaration)
- DialogHeader (dòng 81, function-declaration)
- DialogFooter (dòng 91, function-declaration)
- DialogTitle (dòng 104, function-declaration)
- DialogDescription (dòng 117, function-declaration)

## app_service/src/components/ui/drawer.tsx

- Drawer (dòng 6, function-declaration)
- DrawerTrigger (dòng 12, function-declaration)
- DrawerPortal (dòng 18, function-declaration)
- DrawerClose (dòng 24, function-declaration)
- DrawerOverlay (dòng 30, function-declaration)
- DrawerContent (dòng 46, function-declaration)
- DrawerHeader (dòng 73, function-declaration)
- DrawerFooter (dòng 86, function-declaration)
- DrawerTitle (dòng 96, function-declaration)
- DrawerDescription (dòng 109, function-declaration)

## app_service/src/components/ui/dropdown-menu.tsx

- DropdownMenu (dòng 9, function-declaration)
- DropdownMenuPortal (dòng 15, function-declaration)
- DropdownMenuTrigger (dòng 23, function-declaration)
- DropdownMenuContent (dòng 34, function-declaration)
- DropdownMenuGroup (dòng 54, function-declaration)
- DropdownMenuItem (dòng 62, function-declaration)
- DropdownMenuCheckboxItem (dòng 85, function-declaration)
- DropdownMenuRadioGroup (dòng 111, function-declaration)
- DropdownMenuRadioItem (dòng 122, function-declaration)
- DropdownMenuLabel (dòng 146, function-declaration)
- DropdownMenuSeparator (dòng 166, function-declaration)
- DropdownMenuShortcut (dòng 179, function-declaration)
- DropdownMenuSub (dòng 195, function-declaration)
- DropdownMenuSubTrigger (dòng 201, function-declaration)
- DropdownMenuSubContent (dòng 225, function-declaration)

## app_service/src/components/ui/form.tsx

- useFormField (dòng 43, arrow-function)
- FormItem (dòng 74, function-declaration)
- FormLabel (dòng 88, function-declaration)
- FormControl (dòng 105, function-declaration)
- FormDescription (dòng 123, function-declaration)
- FormMessage (dòng 136, function-declaration)

## app_service/src/components/ui/hover-card.tsx

- HoverCard (dòng 6, function-declaration)
- HoverCardTrigger (dòng 12, function-declaration)
- HoverCardContent (dòng 20, function-declaration)

## app_service/src/components/ui/input-otp.tsx

- InputOTP (dòng 9, function-declaration)
- InputOTPGroup (dòng 29, function-declaration)
- InputOTPSlot (dòng 39, function-declaration)
- InputOTPSeparator (dòng 69, function-declaration)

## app_service/src/components/ui/input.tsx

- Input (dòng 5, function-declaration)

## app_service/src/components/ui/label.tsx

- Label (dòng 8, function-declaration)

## app_service/src/components/ui/menubar.tsx

- Menubar (dòng 7, function-declaration)
- MenubarMenu (dòng 23, function-declaration)
- MenubarGroup (dòng 29, function-declaration)
- MenubarPortal (dòng 35, function-declaration)
- MenubarRadioGroup (dòng 41, function-declaration)
- MenubarTrigger (dòng 49, function-declaration)
- MenubarContent (dòng 65, function-declaration)
- MenubarItem (dòng 89, function-declaration)
- MenubarCheckboxItem (dòng 112, function-declaration)
- MenubarRadioItem (dòng 138, function-declaration)
- MenubarLabel (dòng 162, function-declaration)
- MenubarSeparator (dòng 182, function-declaration)
- MenubarShortcut (dòng 195, function-declaration)
- MenubarSub (dòng 211, function-declaration)
- MenubarSubTrigger (dòng 217, function-declaration)
- MenubarSubContent (dòng 241, function-declaration)

## app_service/src/components/ui/navigation-menu.tsx

- NavigationMenu (dòng 8, function-declaration)
- NavigationMenuList (dòng 32, function-declaration)
- NavigationMenuItem (dòng 48, function-declaration)
- NavigationMenuTrigger (dòng 65, function-declaration)
- NavigationMenuContent (dòng 85, function-declaration)
- NavigationMenuViewport (dòng 102, function-declaration)
- NavigationMenuLink (dòng 124, function-declaration)
- NavigationMenuIndicator (dòng 140, function-declaration)

## app_service/src/components/ui/pagination.tsx

- Pagination (dòng 11, function-declaration)
- PaginationContent (dòng 23, function-declaration)
- PaginationItem (dòng 36, function-declaration)
- PaginationLink (dòng 45, function-declaration)
- PaginationPrevious (dòng 68, function-declaration)
- PaginationNext (dòng 85, function-declaration)
- PaginationEllipsis (dòng 102, function-declaration)

## app_service/src/components/ui/popover.tsx

- Popover (dòng 8, function-declaration)
- PopoverTrigger (dòng 14, function-declaration)
- PopoverContent (dòng 20, function-declaration)
- PopoverAnchor (dòng 42, function-declaration)

## app_service/src/components/ui/progress.tsx

- Progress (dòng 6, function-declaration)

## app_service/src/components/ui/radio-group.tsx

- RadioGroup (dòng 9, function-declaration)
- RadioGroupItem (dòng 22, function-declaration)

## app_service/src/components/ui/resizable.tsx

- ResizablePanelGroup (dòng 7, function-declaration)
- ResizablePanel (dòng 23, function-declaration)
- ResizableHandle (dòng 29, function-declaration)

## app_service/src/components/ui/scroll-area.tsx

- ScrollArea (dòng 8, function-declaration)
- ScrollBar (dòng 31, function-declaration)

## app_service/src/components/ui/select.tsx

- Select (dòng 7, function-declaration)
- SelectGroup (dòng 13, function-declaration)
- SelectValue (dòng 19, function-declaration)
- SelectTrigger (dòng 25, function-declaration)
- SelectContent (dòng 51, function-declaration)
- SelectLabel (dòng 86, function-declaration)
- SelectItem (dòng 99, function-declaration)
- SelectSeparator (dòng 123, function-declaration)
- SelectScrollUpButton (dòng 136, function-declaration)
- SelectScrollDownButton (dòng 154, function-declaration)

## app_service/src/components/ui/separator.tsx

- Separator (dòng 8, function-declaration)

## app_service/src/components/ui/sheet.tsx

- Sheet (dòng 7, function-declaration)
- SheetTrigger (dòng 11, function-declaration)
- SheetClose (dòng 17, function-declaration)
- SheetPortal (dòng 23, function-declaration)
- SheetOverlay (dòng 29, function-declaration)
- SheetContent (dòng 45, function-declaration)
- SheetHeader (dòng 82, function-declaration)
- SheetFooter (dòng 92, function-declaration)
- SheetTitle (dòng 102, function-declaration)
- SheetDescription (dòng 115, function-declaration)

## app_service/src/components/ui/sidebar.tsx

- useSidebar (dòng 47, function-declaration)
- SidebarProvider (dòng 56, function-declaration)
- handleKeyDown (dòng 98, arrow-function)
- Sidebar (dòng 154, function-declaration)
- SidebarTrigger (dòng 256, function-declaration)
- SidebarRail (dòng 282, function-declaration)
- SidebarInset (dòng 307, function-declaration)
- SidebarInput (dòng 321, function-declaration)
- SidebarHeader (dòng 335, function-declaration)
- SidebarFooter (dòng 346, function-declaration)
- SidebarSeparator (dòng 357, function-declaration)
- SidebarContent (dòng 371, function-declaration)
- SidebarGroup (dòng 385, function-declaration)
- SidebarGroupLabel (dòng 396, function-declaration)
- SidebarGroupAction (dòng 417, function-declaration)
- SidebarGroupContent (dòng 440, function-declaration)
- SidebarMenu (dòng 454, function-declaration)
- SidebarMenuItem (dòng 465, function-declaration)
- SidebarMenuButton (dòng 498, function-declaration)
- SidebarMenuAction (dòng 548, function-declaration)
- SidebarMenuBadge (dòng 580, function-declaration)
- SidebarMenuSkeleton (dòng 602, function-declaration)
- SidebarMenuSub (dòng 640, function-declaration)
- SidebarMenuSubItem (dòng 655, function-declaration)
- SidebarMenuSubButton (dòng 669, function-declaration)

## app_service/src/components/ui/skeleton.tsx

- Skeleton (dòng 3, function-declaration)

## app_service/src/components/ui/slider.tsx

- Slider (dòng 8, function-declaration)

## app_service/src/components/ui/sonner.tsx

- Toaster (dòng 4, arrow-function)

## app_service/src/components/ui/switch.tsx

- Switch (dòng 8, function-declaration)

## app_service/src/components/ui/table.tsx

- Table (dòng 5, function-declaration)
- TableHeader (dòng 20, function-declaration)
- TableBody (dòng 30, function-declaration)
- TableFooter (dòng 40, function-declaration)
- TableRow (dòng 53, function-declaration)
- TableHead (dòng 66, function-declaration)
- TableCell (dòng 79, function-declaration)
- TableCaption (dòng 92, function-declaration)

## app_service/src/components/ui/tabs.tsx

- Tabs (dòng 8, function-declaration)
- TabsList (dòng 21, function-declaration)
- TabsTrigger (dòng 37, function-declaration)
- TabsContent (dòng 53, function-declaration)

## app_service/src/components/ui/textarea.tsx

- Textarea (dòng 5, function-declaration)

## app_service/src/components/ui/toggle-group.tsx

- ToggleGroup (dòng 17, function-declaration)
- ToggleGroupItem (dòng 43, function-declaration)

## app_service/src/components/ui/toggle.tsx

- Toggle (dòng 29, function-declaration)

## app_service/src/components/ui/tooltip.tsx

- TooltipProvider (dòng 6, function-declaration)
- Tooltip (dòng 19, function-declaration)
- TooltipTrigger (dòng 29, function-declaration)
- TooltipContent (dòng 35, function-declaration)

## app_service/src/contexts/AuthContext.jsx

- useAuth (dòng 16, arrow-function)
- AuthProvider (dòng 24, arrow-function)

## app_service/src/data/mockData.js

- generateTemperatureData (dòng 3, arrow-function)
- generateHumidityData (dòng 18, arrow-function)
- generateDeviceHistory (dòng 115, arrow-function)
- mockRecentAlerts (dòng 214, arrow-function)
- pick (dòng 216, arrow-function)

## app_service/src/hooks/use-mobile.ts

- useIsMobile (dòng 5, function-declaration)
- onChange (dòng 10, arrow-function)

## app_service/src/lib/deviceStatus.js

- toUiStatus (dòng 12, function-declaration)
- isOnline (dòng 19, function-declaration)

## app_service/src/lib/utils.ts

- cn (dòng 4, function-declaration)

## app_service/src/lib/wsUrl.js

- wsUrl (dòng 12, function-declaration)

## app_service/src/pages/ChangePassword.jsx

- ChangePassword (dòng 6, function-declaration)
- validate (dòng 16, arrow-function)
- handleSubmit (dòng 26, arrow-function)

## app_service/src/pages/DeviceDetail.jsx

- resolveWsBase (dòng 36, function-declaration)
- normalizeDeviceType (dòng 46, function-declaration)
- getDeviceMetrics (dòng 55, function-declaration)
- mapApiDeviceToUi (dòng 79, function-declaration)
- isoDateToDisplay (dòng 97, function-declaration)
- formatTime (dòng 105, function-declaration)
- capPush (dòng 113, function-declaration)
- normalizeEventTsToMs (dòng 122, function-declaration)
- mapHistoryToSeries (dòng 128, function-declaration)
- DeviceDetail (dòng 143, arrow-function)
- load (dòng 172, arrow-function)
- connect (dòng 285, arrow-function)
- onKeydown (dòng 351, arrow-function)
- maskPassword (dòng 388, arrow-function)

## app_service/src/pages/Devices.jsx

- DeviceCardSkeleton (dòng 24, arrow-function)
- normalizeDeviceType (dòng 39, arrow-function)
- DeviceCard (dòng 50, wrapped-component)
- Devices (dòng 158, arrow-function)
- load (dòng 190, arrow-function)
- handleAddDevice (dòng 281, arrow-function)
- handleDeleteDevice (dòng 287, arrow-function)

## app_service/src/pages/Forbidden.jsx

- Forbidden (dòng 5, function-declaration)

## app_service/src/pages/ForgotPassword.jsx

- ForgotPassword (dòng 6, arrow-function)
- validateForm (dòng 13, arrow-function)
- handleSubmit (dòng 22, arrow-function)

## app_service/src/pages/GlobalDashboard.jsx

- resolveWsBase (dòng 17, function-declaration)
- foldText (dòng 27, function-declaration)
- normalizeDeviceType (dòng 35, function-declaration)
- inferDeviceTypeFromUpdate (dòng 43, function-declaration)
- toFiniteNumber (dòng 62, function-declaration)
- normalizeDeviceRow (dòng 67, function-declaration)
- computeYAxisDomain (dòng 78, function-declaration)
- getBarSize (dòng 97, function-declaration)
- getCategoryGap (dòng 102, function-declaration)
- MetricBarChartCard (dòng 116, function-declaration)
- GlobalDashboard (dòng 218, function-declaration)
- loadDevices (dòng 232, arrow-function)
- connect (dòng 317, arrow-function)
- onKeydown (dòng 407, arrow-function)

## app_service/src/pages/GPSPage.jsx

- resolveWsBase (dòng 7, function-declaration)
- GPSPage (dòng 12, arrow-function)
- fetchDeviceCatalog (dòng 21, arrow-function)
- connect (dòng 43, arrow-function)

## app_service/src/pages/gpsRealtime.js

- toFiniteNumber (dòng 1, function-declaration)
- getDeviceId (dòng 6, function-declaration)
- getTimestampIso (dòng 10, function-declaration)
- getGpsUpdates (dòng 20, function-declaration)
- mergeGpsMessage (dòng 42, function-declaration)
- mergeDeviceCatalog (dòng 76, function-declaration)

## app_service/src/pages/Home.jsx

- Home (dòng 9, function-declaration)
- statusLabel (dòng 12, arrow-function)

## app_service/src/pages/Login.jsx

- Login (dòng 6, arrow-function)
- validateForm (dòng 14, arrow-function)
- handleSubmit (dòng 35, arrow-function)

## app_service/src/pages/TopicManagement.jsx

- normalizeDevices (dòng 5, function-declaration)
- TopicManagement (dòng 15, function-declaration)
- loadAll (dòng 25, arrow-function)
- saveTopic (dòng 61, arrow-function)

## app_service/src/pages/UserManagement.jsx

- defaultExpiredAt (dòng 7, function-declaration)
- isoToDdMmYyyy (dòng 14, function-declaration)
- ddMmYyyyToIso (dòng 22, function-declaration)
- todayIso (dòng 37, function-declaration)
- emptyForm (dòng 41, arrow-function)
- UserManagement (dòng 53, function-declaration)
- openModal (dòng 144, arrow-function)
- handleStatusChange (dòng 153, arrow-function)
- handleRegister (dòng 169, arrow-function)
- openDelete (dòng 221, arrow-function)
- openAssign (dòng 227, arrow-function)
- handleDelete (dòng 231, arrow-function)
