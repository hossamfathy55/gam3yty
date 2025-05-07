# تصميم نماذج البيانات (Data Models) لتطبيق جمعيتي (Jam3yeti Savings Buddy) - قاعدة بيانات MySQL

## 1. جدول المستخدمين (Users)

*   **اسم الجدول:** `users`
*   **الوصف:** يخزن معلومات المستخدمين المسجلين في التطبيق.
*   **الحقول:**
    *   `id`: INT, PRIMARY KEY, AUTO_INCREMENT, NOT NULL (معرف المستخدم)
    *   `username`: VARCHAR(255), UNIQUE, NOT NULL (اسم المستخدم للدخول)
    *   `email`: VARCHAR(255), UNIQUE, NOT NULL (البريد الإلكتروني للمستخدم)
    *   `password_hash`: VARCHAR(255), NOT NULL (كلمة المرور المخزنة بشكل آمن - Hashed)
    *   `full_name`: VARCHAR(255), NOT NULL (الاسم الكامل للمستخدم)
    *   `created_at`: TIMESTAMP, DEFAULT CURRENT_TIMESTAMP (تاريخ إنشاء الحساب)
    *   `updated_at`: TIMESTAMP, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP (تاريخ آخر تحديث للحساب)
    *   `is_active`: BOOLEAN, DEFAULT TRUE (لتحديد ما إذا كان الحساب نشطاً)
    *   `last_login_at`: TIMESTAMP, NULLABLE (تاريخ آخر تسجيل دخول)

## 2. جدول المجموعات (Groups/Jam3yat)

*   **اسم الجدول:** `groups`
*   **الوصف:** يخزن معلومات مجموعات الادخار (الجمعيات).
*   **الحقول:**
    *   `id`: INT, PRIMARY KEY, AUTO_INCREMENT, NOT NULL (معرف المجموعة)
    *   `name`: VARCHAR(255), NOT NULL (اسم المجموعة)
    *   `description`: TEXT, NULLABLE (وصف المجموعة)
    *   `contribution_amount`: DECIMAL(10, 2), NOT NULL (مبلغ المساهمة المطلوب من كل عضو)
    *   `payout_frequency`: ENUM('weekly', 'monthly', 'bi-monthly'), NOT NULL (تكرار دورة الدفع)
    *   `max_members`: INT, NOT NULL (الحد الأقصى لعدد الأعضاء)
    *   `current_members_count`: INT, DEFAULT 0 (عدد الأعضاء الحاليين)
    *   `creator_user_id`: INT, NOT NULL (معرف المستخدم الذي أنشأ المجموعة)
    *   `status`: ENUM('pending', 'open_for_joining', 'active', 'completed', 'cancelled'), DEFAULT 'pending' (حالة المجموعة)
    *   `start_date`: DATE, NULLABLE (تاريخ بدء دورات المجموعة فعلياً)
    *   `created_at`: TIMESTAMP, DEFAULT CURRENT_TIMESTAMP (تاريخ إنشاء المجموعة)
    *   `updated_at`: TIMESTAMP, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP (تاريخ آخر تحديث للمجموعة)
    *   **العلاقات:**
        *   FOREIGN KEY (`creator_user_id`) REFERENCES `users`(`id`)

## 3. جدول العضويات (Memberships)

*   **اسم الجدول:** `memberships`
*   **الوصف:** يربط المستخدمين بالمجموعات التي هم أعضاء فيها (جدول وسيط Many-to-Many).
*   **الحقول:**
    *   `id`: INT, PRIMARY KEY, AUTO_INCREMENT, NOT NULL (معرف العضوية)
    *   `user_id`: INT, NOT NULL (معرف المستخدم العضو)
    *   `group_id`: INT, NOT NULL (معرف المجموعة)
    *   `join_date`: TIMESTAMP, DEFAULT CURRENT_TIMESTAMP (تاريخ الانضمام)
    *   `status`: ENUM('pending_approval', 'active', 'left', 'removed'), DEFAULT 'pending_approval' (حالة العضوية)
    *   `payout_order_preference`: INT, NULLABLE (ترتيب الأفضلية لاستلام الدفعة، إذا كان مسموحاً به)
    *   `is_admin`: BOOLEAN, DEFAULT FALSE (هل المستخدم هو مسؤول المجموعة - قد يكون المنشئ أو عضو آخر تم ترقيته)
    *   **العلاقات:**
        *   FOREIGN KEY (`user_id`) REFERENCES `users`(`id`)
        *   FOREIGN KEY (`group_id`) REFERENCES `groups`(`id`)
        *   UNIQUE KEY (`user_id`, `group_id`) (لضمان عدم انضمام المستخدم لنفس المجموعة أكثر من مرة)

## 4. جدول المساهمات (Contributions)

*   **اسم الجدول:** `contributions`
*   **الوصف:** يخزن سجلات مساهمات الأعضاء في المجموعات.
*   **الحقول:**
    *   `id`: INT, PRIMARY KEY, AUTO_INCREMENT, NOT NULL (معرف المساهمة)
    *   `membership_id`: INT, NOT NULL (معرف العضوية المرتبطة بالمساهمة)
    *   `group_id`: INT, NOT NULL (معرف المجموعة للتسهيل)
    *   `user_id`: INT, NOT NULL (معرف المستخدم المساهم للتسهيل)
    *   `amount`: DECIMAL(10, 2), NOT NULL (مبلغ المساهمة)
    *   `contribution_date`: DATE, NOT NULL (تاريخ المساهمة)
    *   `payment_cycle_identifier`: VARCHAR(255), NOT NULL (معرف دورة الدفع، مثلاً "2024-May")
    *   `payment_method`: VARCHAR(255), NULLABLE (طريقة الدفع المستخدمة)
    *   `transaction_reference`: VARCHAR(255), NULLABLE (مرجع المعاملة إذا وجد)
    *   `status`: ENUM('pending', 'confirmed', 'late', 'waived'), DEFAULT 'pending' (حالة المساهمة)
    *   `created_at`: TIMESTAMP, DEFAULT CURRENT_TIMESTAMP
    *   **العلاقات:**
        *   FOREIGN KEY (`membership_id`) REFERENCES `memberships`(`id`)
        *   FOREIGN KEY (`group_id`) REFERENCES `groups`(`id`)
        *   FOREIGN KEY (`user_id`) REFERENCES `users`(`id`)

## 5. جدول دورات الدفع/الاستحقاقات (Payouts)

*   **اسم الجدول:** `payouts`
*   **الوصف:** يخزن سجلات عمليات الدفع للأعضاء المستحقين في كل دورة.
*   **الحقول:**
    *   `id`: INT, PRIMARY KEY, AUTO_INCREMENT, NOT NULL (معرف عملية الدفع)
    *   `group_id`: INT, NOT NULL (معرف المجموعة)
    *   `receiving_membership_id`: INT, NOT NULL (معرف عضوية المستخدم الذي استلم الدفعة)
    *   `payout_cycle_identifier`: VARCHAR(255), NOT NULL (معرف دورة الدفع، مثلاً "2024-May")
    *   `total_amount_collected`: DECIMAL(10, 2), NOT NULL (إجمالي المبلغ المجمع لهذه الدورة)
    *   `payout_amount`: DECIMAL(10, 2), NOT NULL (المبلغ المدفوع للعضو)
    *   `payout_date`: DATE, NOT NULL (تاريخ الدفع)
    *   `status`: ENUM('scheduled', 'paid', 'failed'), DEFAULT 'scheduled' (حالة عملية الدفع)
    *   `notes`: TEXT, NULLABLE (ملاحظات إضافية)
    *   `created_at`: TIMESTAMP, DEFAULT CURRENT_TIMESTAMP
    *   **العلاقات:**
        *   FOREIGN KEY (`group_id`) REFERENCES `groups`(`id`)
        *   FOREIGN KEY (`receiving_membership_id`) REFERENCES `memberships`(`id`)

## 6. جدول رموز إعادة تعيين كلمة المرور (PasswordResetTokens)

*   **اسم الجدول:** `password_reset_tokens`
*   **الوصف:** يخزن الرموز المؤقتة المستخدمة لعملية إعادة تعيين كلمة المرور.
*   **الحقول:**
    *   `id`: INT, PRIMARY KEY, AUTO_INCREMENT, NOT NULL
    *   `user_id`: INT, NOT NULL
    *   `token`: VARCHAR(255), UNIQUE, NOT NULL (الرمز الفريد)
    *   `expires_at`: TIMESTAMP, NOT NULL (تاريخ انتهاء صلاحية الرمز)
    *   `created_at`: TIMESTAMP, DEFAULT CURRENT_TIMESTAMP
    *   **العلاقات:**
        *   FOREIGN KEY (`user_id`) REFERENCES `users`(`id`) ON DELETE CASCADE

## 7. جدول الإشعارات (Notifications) - (مستقبلي، لكن مهم للتخطيط)

*   **اسم الجدول:** `notifications`
*   **الوصف:** يخزن الإشعارات للمستخدمين.
*   **الحقول:**
    *   `id`: INT, PRIMARY KEY, AUTO_INCREMENT, NOT NULL
    *   `user_id`: INT, NOT NULL (المستخدم المستلم للإشعار)
    *   `type`: VARCHAR(255), NOT NULL (نوع الإشعار، مثل 'new_group_invite', 'payment_due', 'payout_received')
    *   `message`: TEXT, NOT NULL (محتوى الإشعار)
    *   `related_entity_type`: VARCHAR(255), NULLABLE (نوع الكيان المرتبط، مثل 'group', 'contribution')
    *   `related_entity_id`: INT, NULLABLE (معرف الكيان المرتبط)
    *   `is_read`: BOOLEAN, DEFAULT FALSE
    *   `created_at`: TIMESTAMP, DEFAULT CURRENT_TIMESTAMP
    *   **العلاقات:**
        *   FOREIGN KEY (`user_id`) REFERENCES `users`(`id`)

## ملاحظات عامة على التصميم:

*   **الفهارس (Indexes):** سيتم إضافة فهارس مناسبة للحقول التي يتم البحث أو الفرز بناءً عليها بشكل متكرر لتحسين الأداء (مثل `email` في `users`, `status` في `groups`, `user_id` و `group_id` في `memberships`).
*   **السلامة المرجعية (Referential Integrity):** سيتم فرضها باستخدام المفاتيح الأجنبية.
*   **أنواع البيانات:** تم اختيار أنواع بيانات MySQL القياسية. قد تحتاج بعض الحقول للمراجعة بناءً على متطلبات دقيقة (مثل طول VARCHARs).
*   **التطبيع (Normalization):** تم تصميم الجداول مع مراعاة مبادئ التطبيع لتقليل تكرار البيانات.
*   **التوسع المستقبلي:** تم تضمين بعض الحقول أو الجداول (مثل `Notifications`) التي قد تكون مفيدة للتوسع المستقبلي.

