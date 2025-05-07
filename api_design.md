# تصميم الواجهة البرمجية (API Endpoints) لتطبيق جمعيتي (Jam3yeti Savings Buddy)

## 1. مصادقة المستخدمين (Authentication)

*   **POST /api/auth/register**
    *   الوصف: تسجيل مستخدم جديد.
    *   المدخلات (Request Body): `username`, `email`, `password`, `full_name`
    *   المخرجات (Response): `user_id`, `message`
*   **POST /api/auth/login**
    *   الوصف: تسجيل دخول مستخدم حالي.
    *   المدخلات (Request Body): `email`, `password`
    *   المخرجات (Response): `access_token`, `refresh_token`, `user_details`
*   **POST /api/auth/logout**
    *   الوصف: تسجيل خروج المستخدم (إبطال التوكن).
    *   المدخلات (Headers): `Authorization: Bearer <access_token>`
    *   المخرجات (Response): `message`
*   **POST /api/auth/refresh-token**
    *   الوصف: طلب توكن وصول جديد باستخدام توكن التحديث.
    *   المدخلات (Request Body): `refresh_token`
    *   المخرجات (Response): `access_token`
*   **POST /api/auth/forgot-password**
    *   الوصف: طلب إعادة تعيين كلمة المرور.
    *   المدخلات (Request Body): `email`
    *   المخرجات (Response): `message`
*   **POST /api/auth/reset-password**
    *   الوصف: إعادة تعيين كلمة المرور باستخدام رمز.
    *   المدخلات (Request Body): `reset_token`, `new_password`
    *   المخرجات (Response): `message`

## 2. إدارة المستخدمين (Users)

*   **GET /api/users/me**
    *   الوصف: الحصول على تفاصيل المستخدم الحالي.
    *   المدخلات (Headers): `Authorization: Bearer <access_token>`
    *   المخرجات (Response): `user_details` (id, username, email, full_name, etc.)
*   **PUT /api/users/me**
    *   الوصف: تحديث تفاصيل المستخدم الحالي.
    *   المدخلات (Headers): `Authorization: Bearer <access_token>`
    *   المدخلات (Request Body): `full_name`, `email` (وغيرها من الحقول القابلة للتعديل)
    *   المخرجات (Response): `updated_user_details`
*   **PUT /api/users/me/change-password**
    *   الوصف: تغيير كلمة مرور المستخدم الحالي.
    *   المدخلات (Headers): `Authorization: Bearer <access_token>`
    *   المدخلات (Request Body): `current_password`, `new_password`
    *   المخرجات (Response): `message`

## 3. إدارة المجموعات (Jam3yat/Groups)

*   **POST /api/groups**
    *   الوصف: إنشاء مجموعة جديدة.
    *   المدخلات (Headers): `Authorization: Bearer <access_token>`
    *   المدخلات (Request Body): `name`, `description`, `contribution_amount`, `payout_frequency` (e.g., monthly), `max_members`, `start_date`
    *   المخرجات (Response): `group_details`
*   **GET /api/groups**
    *   الوصف: الحصول على قائمة بجميع المجموعات المتاحة (مع إمكانية الفلترة والترتيب).
    *   المدخلات (Query Params): `status` (e.g., open, active, closed), `sort_by`, `order`
    *   المخرجات (Response): `list_of_groups`
*   **GET /api/groups/{groupId}**
    *   الوصف: الحصول على تفاصيل مجموعة معينة.
    *   المدخلات (Path Params): `groupId`
    *   المخرجات (Response): `group_details` (including members, contribution status, etc.)
*   **PUT /api/groups/{groupId}**
    *   الوصف: تحديث تفاصيل مجموعة (فقط لمنشئ المجموعة أو المسؤول).
    *   المدخلات (Headers): `Authorization: Bearer <access_token>`
    *   المدخلات (Path Params): `groupId`
    *   المدخلات (Request Body): `name`, `description`, etc.
    *   المخرجات (Response): `updated_group_details`
*   **DELETE /api/groups/{groupId}**
    *   الوصف: حذف مجموعة (فقط لمنشئ المجموعة، بشروط معينة).
    *   المدخلات (Headers): `Authorization: Bearer <access_token>`
    *   المدخلات (Path Params): `groupId`
    *   المخرجات (Response): `message`
*   **POST /api/groups/{groupId}/join**
    *   الوصف: طلب الانضمام إلى مجموعة.
    *   المدخلات (Headers): `Authorization: Bearer <access_token>`
    *   المدخلات (Path Params): `groupId`
    *   المخرجات (Response): `message` (e.g., request sent, or joined successfully if auto-approve)
*   **POST /api/groups/{groupId}/leave**
    *   الوصف: مغادرة مجموعة.
    *   المدخلات (Headers): `Authorization: Bearer <access_token>`
    *   المدخلات (Path Params): `groupId`
    *   المخرجات (Response): `message`
*   **GET /api/users/me/groups**
    *   الوصف: الحصول على قائمة المجموعات التي ينتمي إليها المستخدم الحالي.
    *   المدخلات (Headers): `Authorization: Bearer <access_token>`
    *   المخرجات (Response): `list_of_my_groups`

## 4. إدارة الأعضاء في المجموعة (Group Members Management - by Admin)

*   **GET /api/groups/{groupId}/members**
    *   الوصف: الحصول على قائمة أعضاء المجموعة (لمسؤول المجموعة).
    *   المدخلات (Headers): `Authorization: Bearer <access_token>`
    *   المدخلات (Path Params): `groupId`
    *   المخرجات (Response): `list_of_members`
*   **POST /api/groups/{groupId}/members/{userId}/approve**
    *   الوصف: الموافقة على طلب انضمام عضو (لمسؤول المجموعة).
    *   المدخلات (Headers): `Authorization: Bearer <access_token>`
    *   المدخلات (Path Params): `groupId`, `userId`
    *   المخرجات (Response): `message`
*   **POST /api/groups/{groupId}/members/{userId}/reject**
    *   الوصف: رفض طلب انضمام عضو (لمسؤول المجموعة).
    *   المدخلات (Headers): `Authorization: Bearer <access_token>`
    *   المدخلات (Path Params): `groupId`, `userId`
    *   المخرجات (Response): `message`
*   **DELETE /api/groups/{groupId}/members/{userId}**
    *   الوصف: إزالة عضو من المجموعة (لمسؤول المجموعة).
    *   المدخلات (Headers): `Authorization: Bearer <access_token>`
    *   المدخلات (Path Params): `groupId`, `userId`
    *   المخرجات (Response): `message`

## 5. إدارة المساهمات/الدفعات (Contributions/Payments)

*   **POST /api/groups/{groupId}/contributions**
    *   الوصف: تسجيل دفعة مساهمة من قبل عضو في المجموعة.
    *   المدخلات (Headers): `Authorization: Bearer <access_token>`
    *   المدخلات (Path Params): `groupId`
    *   المدخلات (Request Body): `amount`, `payment_date`, `payment_method_details` (optional)
    *   المخرجات (Response): `contribution_details`
*   **GET /api/groups/{groupId}/contributions**
    *   الوصف: الحصول على قائمة بجميع المساهمات في مجموعة معينة (لمسؤول المجموعة أو الأعضاء).
    *   المدخلات (Headers): `Authorization: Bearer <access_token>`
    *   المدخلات (Path Params): `groupId`
    *   المخرجات (Response): `list_of_contributions`
*   **GET /api/users/me/contributions**
    *   الوصف: الحصول على تاريخ مساهمات المستخدم الحالي.
    *   المدخلات (Headers): `Authorization: Bearer <access_token>`
    *   المخرجات (Response): `list_of_my_contributions`

## 6. إدارة دورات الدفع (Payout Cycles - for Admin)

*   **POST /api/groups/{groupId}/payouts**
    *   الوصف: تسجيل عملية دفع لعضو مستحق في دورة حالية (لمسؤول المجموعة).
    *   المدخلات (Headers): `Authorization: Bearer <access_token>`
    *   المدخلات (Path Params): `groupId`
    *   المدخلات (Request Body): `member_id_receiving_payout`, `payout_date`, `amount_paid`
    *   المخرجات (Response): `payout_details`
*   **GET /api/groups/{groupId}/payouts**
    *   الوصف: الحصول على تاريخ عمليات الدفع للمجموعة (لمسؤول المجموعة أو الأعضاء).
    *   المدخلات (Headers): `Authorization: Bearer <access_token>`
    *   المدخلات (Path Params): `groupId`
    *   المخرجات (Response): `list_of_payouts`

## ملاحظات عامة:

*   جميع الطلبات التي تتطلب مصادقة يجب أن تتضمن `Authorization: Bearer <access_token>` في الترويسة.
*   يجب معالجة الأخطاء بشكل مناسب مع رموز حالة HTTP واضحة ورسائل خطأ مفهومة.
*   سيتم استخدام HTTPS لجميع الاتصالات.
*   سيتم تطبيق التحقق من صحة المدخلات على مستوى الخادم لجميع الطلبات.

