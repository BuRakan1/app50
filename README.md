    def _open_faculty_paths_page(self):
        """فتح صفحة مسارات هيئة التدريس مع عرض المواد كبطاقات"""
        faculty_window = tk.Toplevel(self)
        faculty_window.title("مسارات هيئة التدريس")
        faculty_window.state('zoomed')  # ملء الشاشة
        faculty_window.configure(bg="#F5F5F5")
        faculty_window.transient(self)
        faculty_window.grab_set()

        # شريط العنوان
        header_frame = tk.Frame(faculty_window, bg="#1E3A5F", height=100)
        header_frame.pack(fill=tk.X)
        header_frame.pack_propagate(False)

        header_content = tk.Frame(header_frame, bg="#1E3A5F")
        header_content.pack(expand=True, fill=tk.BOTH)

        # العنوان في المنتصف
        title_container = tk.Frame(header_content, bg="#1E3A5F")
        title_container.place(relx=0.5, rely=0.5, anchor="center")

        title_label = tk.Label(
            title_container,
            text="مسارات هيئة التدريس",
            font=("Tajawal", 32, "bold"),
            bg="#1E3A5F",
            fg="white"
        )
        title_label.pack()

        # الأزرار على اليمين
        buttons_container = tk.Frame(header_content, bg="#1E3A5F")
        buttons_container.place(relx=0.98, rely=0.5, anchor="e")

        # زر تصدير البيانات
        export_btn = tk.Button(
            buttons_container,
            text="📊 تصدير بيانات Excel",
            font=("Tajawal", 14, "bold"),
            bg="#4CAF50",
            fg="white",
            padx=20,
            pady=10,
            bd=0,
            relief=tk.FLAT,
            cursor="hand2",
            command=lambda: self._export_faculty_paths_data()
        )
        export_btn.pack(side=tk.LEFT, padx=(0, 10))

        # زر الإغلاق
        close_btn = tk.Button(
            buttons_container,
            text="✕ إغلاق",
            font=("Tajawal", 14, "bold"),
            bg="#dc3545",
            fg="white",
            padx=20,
            pady=10,
            bd=0,
            relief=tk.FLAT,
            cursor="hand2",
            command=faculty_window.destroy
        )
        close_btn.pack(side=tk.LEFT)

        # إطار البحث
        search_frame = tk.Frame(faculty_window, bg=self.COLORS["surface"], height=60)
        search_frame.pack(fill=tk.X, padx=20, pady=(10, 0))
        search_frame.pack_propagate(False)

        search_container = tk.Frame(search_frame, bg=self.COLORS["surface"])
        search_container.pack(side=tk.LEFT, padx=20, pady=15)

        tk.Label(
            search_container,
            text="البحث عن مسار:",
            font=("Tajawal", 14),
            bg=self.COLORS["surface"]
        ).pack(side=tk.LEFT, padx=(0, 10))

        search_entry = tk.Entry(
            search_container,
            font=("Tajawal", 13),
            width=40,
            bd=2,
            relief=tk.FLAT,
            highlightthickness=1,
            highlightcolor="#1E3A5F"
        )
        search_entry.pack(side=tk.LEFT)

        # إطار رئيسي قابل للتمرير
        main_container = tk.Frame(faculty_window, bg="#F5F5F5")
        main_container.pack(fill=tk.BOTH, expand=True, padx=20, pady=20)

        # Canvas للتمرير
        canvas = tk.Canvas(main_container, bg="#F5F5F5", highlightthickness=0)
        v_scrollbar = ttk.Scrollbar(main_container, orient="vertical", command=canvas.yview)
        h_scrollbar = ttk.Scrollbar(main_container, orient="horizontal", command=canvas.xview)

        scrollable_frame = tk.Frame(canvas, bg="#F5F5F5")

        scrollable_frame.bind(
            "<Configure>",
            lambda e: canvas.configure(scrollregion=canvas.bbox("all"))
        )

        canvas.create_window((0, 0), window=scrollable_frame, anchor="nw")
        canvas.configure(yscrollcommand=v_scrollbar.set, xscrollcommand=h_scrollbar.set)

        # إطار المحتوى للبطاقات
        content_frame = tk.Frame(scrollable_frame, bg="#F5F5F5")
        content_frame.pack(fill=tk.BOTH, expand=True, padx=20, pady=20)

        # دالة لعرض الأعضاء
        def show_members(subject_name, member_type):
            """عرض أعضاء المسار في نافذة منفصلة بتصميم أنيق ومختصر"""
            members_window = tk.Toplevel(faculty_window)
            members_window.title(f"{member_type} - مسار {subject_name}")
            members_window.geometry("1000x700")
            members_window.configure(bg="#FAFAFA")
            members_window.transient(faculty_window)
            members_window.grab_set()

            # توسيط النافذة
            members_window.update_idletasks()
            x = (members_window.winfo_screenwidth() - 1000) // 2
            y = (members_window.winfo_screenheight() - 700) // 2
            members_window.geometry(f"1000x700+{x}+{y}")

            # شريط العنوان
            header = tk.Frame(members_window, bg="#1E3A5F", height=80)
            header.pack(fill=tk.X)
            header.pack_propagate(False)

            header_inner = tk.Frame(header, bg="#1E3A5F")
            header_inner.pack(expand=True)

            tk.Label(
                header_inner,
                text=f"{member_type} - مسار {subject_name}",
                font=("Tajawal", 22, "bold"),
                bg="#1E3A5F",
                fg="white"
            ).pack()

            # زر الإغلاق
            close_members_btn = tk.Button(
                header,
                text="✕ إغلاق",
                font=("Tajawal", 12, "bold"),
                bg="#dc3545",
                fg="white",
                padx=25,
                pady=8,
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                command=members_window.destroy
            )
            close_members_btn.place(relx=0.95, rely=0.5, anchor="e")

            # إطار المحتوى
            content = tk.Frame(members_window, bg="#FAFAFA")
            content.pack(fill=tk.BOTH, expand=True, padx=30, pady=20)

            # الحصول على البيانات من قاعدة البيانات
            cursor = self.db_conn.cursor()

            # رئيس المسار
            cursor.execute("""
                SELECT t.name, t.rank, t.phone
                FROM faculty_subject_paths fsp
                JOIN teachers t ON fsp.teacher_id = t.id
                WHERE fsp.subject_name = ? AND fsp.is_head = 1
            """, (subject_name,))

            head = cursor.fetchone()

            if head:
                # إطار رئيس المسار بتصميم مبسط
                head_frame = tk.Frame(content, bg="#FFF8DC", relief=tk.RAISED, bd=2)
                head_frame.pack(fill=tk.X, pady=(0, 30))

                # شريط علوي ذهبي
                gold_bar = tk.Frame(head_frame, bg="#FFD700", height=5)
                gold_bar.pack(fill=tk.X)

                head_inner = tk.Frame(head_frame, bg="#FFF8DC", padx=20, pady=15)
                head_inner.pack(fill=tk.BOTH)

                tk.Label(
                    head_inner,
                    text="رئيس المسار",
                    font=("Tajawal", 18, "bold"),
                    bg="#FFF8DC",
                    fg="#8B4513"
                ).pack(pady=(0, 10))

                # معلومات الرئيس
                tk.Label(
                    head_inner,
                    text=f"{head[1]} - {head[0]}",
                    font=("Tajawal", 16, "bold"),
                    bg="#FFF8DC",
                    fg="#2F4F4F"
                ).pack()

                if head[2]:
                    tk.Label(
                        head_inner,
                        text=f"📱 {head[2]}",
                        font=("Tajawal", 14),
                        bg="#FFF8DC",
                        fg="#696969"
                    ).pack(pady=(5, 0))

            # خط فاصل
            separator_frame = tk.Frame(content, bg="#FAFAFA", height=20)
            separator_frame.pack(fill=tk.X)

            separator_line = tk.Frame(separator_frame, bg="#E0E0E0", height=2)
            separator_line.pack(expand=True, fill=tk.X, padx=50)

            # الأعضاء
            if member_type == "منسوبي المدينة":
                cursor.execute("""
                    SELECT t.name, t.rank, t.phone
                    FROM faculty_subject_paths fsp
                    JOIN teachers t ON fsp.teacher_id = t.id
                    WHERE fsp.subject_name = ? AND fsp.is_head = 0
                    AND t.category = 'منسوبي المدينة'
                    ORDER BY t.name
                """, (subject_name,))
            else:  # المتعاونين
                cursor.execute("""
                    SELECT t.name, t.rank, t.phone, t.category
                    FROM faculty_subject_paths fsp
                    JOIN teachers t ON fsp.teacher_id = t.id
                    WHERE fsp.subject_name = ? AND fsp.is_head = 0
                    AND t.category IN ('متعاون', 'متعاون مدني', 'متعاون عسكري', 'متعاون عسكري متقاعد')
                    ORDER BY t.name
                """, (subject_name,))

            members = cursor.fetchall()

            if members:
                # عنوان الأعضاء
                members_title_frame = tk.Frame(content, bg="#FAFAFA")
                members_title_frame.pack(pady=(0, 20))

                tk.Label(
                    members_title_frame,
                    text=f"أعضاء المسار - {member_type}",
                    font=("Tajawal", 20, "bold"),
                    bg="#FAFAFA",
                    fg="#1E3A5F"
                ).pack()

                tk.Label(
                    members_title_frame,
                    text=f"العدد الإجمالي: {len(members)}",
                    font=("Tajawal", 14),
                    bg="#FAFAFA",
                    fg="#666666"
                ).pack(pady=5)

                # إطار الأعضاء
                members_container = tk.Frame(content, bg="#FAFAFA")
                members_container.pack(fill=tk.BOTH, expand=True)

                # عرض الأعضاء في شبكة
                columns = 3
                for idx, member in enumerate(members):
                    row = idx // columns
                    col = idx % columns

                    # بطاقة العضو بتصميم مبسط
                    member_card = tk.Frame(
                        members_container,
                        bg="white",
                        relief=tk.RAISED,
                        bd=1
                    )
                    member_card.grid(row=row, column=col, padx=15, pady=15, sticky="nsew")

                    # شريط علوي ملون
                    if member_type == "منسوبي المدينة":
                        top_bar_color = "#2E7D32"
                    else:
                        top_bar_color = "#1565C0"

                    top_bar = tk.Frame(member_card, bg=top_bar_color, height=4)
                    top_bar.pack(fill=tk.X)

                    # محتوى البطاقة
                    card_inner = tk.Frame(member_card, bg="white", padx=20, pady=15)
                    card_inner.pack(fill=tk.BOTH, expand=True)

                    # الاسم والرتبة
                    tk.Label(
                        card_inner,
                        text=f"{member[1]} - {member[0]}",
                        font=("Tajawal", 14, "bold"),
                        bg="white",
                        fg="#1E3A5F",
                        wraplength=250,
                        justify=tk.CENTER
                    ).pack(pady=(0, 10))

                    # رقم الجوال
                    if member[2]:
                        tk.Label(
                            card_inner,
                            text=f"📱 {member[2]}",
                            font=("Tajawal", 12),
                            bg="white",
                            fg="#666666"
                        ).pack()

                    # نوع المتعاون
                    if member_type == "المتعاونين" and len(member) > 3:
                        tk.Label(
                            card_inner,
                            text=f"{member[3]}",
                            font=("Tajawal", 11, "italic"),
                            bg="white",
                            fg="#999999"
                        ).pack(pady=(5, 0))

                    # إضافة تأثير hover
                    def on_enter(e, card=member_card):
                        card.config(bd=2)

                    def on_leave(e, card=member_card):
                        card.config(bd=1)

                    member_card.bind("<Enter>", on_enter)
                    member_card.bind("<Leave>", on_leave)

                # تكوين الأعمدة
                for i in range(columns):
                    members_container.grid_columnconfigure(i, weight=1, minsize=280)

            else:
                # رسالة عدم وجود أعضاء
                no_members_frame = tk.Frame(content, bg="#FFF3E0", relief=tk.RIDGE, bd=2)
                no_members_frame.pack(pady=50, padx=100)

                tk.Label(
                    no_members_frame,
                    text="📭",
                    font=("Arial", 48),
                    bg="#FFF3E0"
                ).pack(pady=20)

                tk.Label(
                    no_members_frame,
                    text=f"لا يوجد {member_type} مسجلين في هذا المسار حالياً",
                    font=("Tajawal", 16),
                    bg="#FFF3E0",
                    fg="#795548"
                ).pack(padx=40, pady=(0, 20))

        # دالة لإدارة المسار
        def manage_path(subject_name):
            """فتح نافذة إدارة المسار بنفس تصميم manage_course_teachers"""
            manage_window = tk.Toplevel(faculty_window)
            manage_window.title(f"إدارة مسار - {subject_name}")
            manage_window.geometry("1200x750")
            manage_window.configure(bg="#F5F5F5")
            manage_window.resizable(True, True)
            manage_window.minsize(1100, 700)

            # توسيط النافذة
            manage_window.update_idletasks()
            x = (manage_window.winfo_screenwidth() - 1200) // 2
            y = (manage_window.winfo_screenheight() - 750) // 2
            manage_window.geometry(f"1200x750+{x}+{y}")

            # شريط العنوان الرئيسي
            header_frame = tk.Frame(manage_window, bg="#1E3A5F", height=70)
            header_frame.pack(fill=tk.X)
            header_frame.pack_propagate(False)

            header_content = tk.Frame(header_frame, bg="#1E3A5F")
            header_content.pack(expand=True)

            tk.Label(
                header_content,
                text="إدارة أعضاء المسار",
                font=("Tajawal", 20, "bold"),
                bg="#1E3A5F",
                fg="white"
            ).pack()

            tk.Label(
                header_content,
                text=f"مسار: {subject_name}",
                font=("Tajawal", 14),
                bg="#1E3A5F",
                fg="#E0E0E0"
            ).pack()

            # إطار المحتوى الرئيسي
            main_container = tk.Frame(manage_window, bg="#F5F5F5")
            main_container.pack(fill=tk.BOTH, expand=True, padx=20, pady=15)

            # القسم الأول: رئيس المسار
            responsible_frame = tk.Frame(main_container, bg="#FFFFFF", bd=1, relief=tk.RIDGE)
            responsible_frame.pack(fill=tk.X, pady=(0, 15))

            # رأس قسم رئيس المسار
            resp_header = tk.Frame(responsible_frame, bg="#2C3E50", height=40)
            resp_header.pack(fill=tk.X)
            resp_header.pack_propagate(False)

            tk.Label(
                resp_header,
                text=f"رئيس مسار ({subject_name})",
                font=("Tajawal", 14, "bold"),
                bg="#2C3E50",
                fg="white"
            ).pack(expand=True)

            # محتوى قسم رئيس المسار
            resp_content = tk.Frame(responsible_frame, bg="#FFFFFF", padx=20, pady=15)
            resp_content.pack(fill=tk.BOTH)

            # تقسيم أفقي
            resp_content.grid_columnconfigure(0, weight=1)
            resp_content.grid_columnconfigure(1, weight=1)

            # الجانب الأيمن - الرئيس الحالي
            right_resp = tk.Frame(resp_content, bg="#FFFFFF")
            right_resp.grid(row=0, column=1, sticky="nsew", padx=(10, 0))

            tk.Label(
                right_resp,
                text="الرئيس الحالي",
                font=("Tajawal", 13, "bold"),
                bg="#FFFFFF",
                fg="#2C3E50"
            ).pack(anchor=tk.E, pady=(0, 10))

            # إطار عرض الرئيس
            current_resp_frame = tk.Frame(right_resp, bg="#F8F9FA", relief=tk.RIDGE, bd=1, height=70)
            current_resp_frame.pack(fill=tk.X)
            current_resp_frame.pack_propagate(False)

            current_responsible_label = tk.Label(
                current_resp_frame,
                text="",
                font=("Tajawal", 12),
                bg="#F8F9FA",
                fg="#333333",
                justify=tk.CENTER
            )
            current_responsible_label.pack(expand=True)

            # زر حذف الرئيس
            remove_resp_btn = tk.Button(
                right_resp,
                text="حذف الرئيس",
                font=("Tajawal", 11, "bold"),
                bg="#B03A2E",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=15,
                pady=6
            )
            remove_resp_btn.pack(pady=(10, 0))

            # الجانب الأيسر - البحث وتعيين
            left_resp = tk.Frame(resp_content, bg="#FFFFFF")
            left_resp.grid(row=0, column=0, sticky="nsew", padx=(0, 10))

            tk.Label(
                left_resp,
                text="تعيين رئيس جديد",
                font=("Tajawal", 13, "bold"),
                bg="#FFFFFF",
                fg="#2C3E50"
            ).pack(anchor=tk.E, pady=(0, 10))

            # البحث
            search_container = tk.Frame(left_resp, bg="#FFFFFF")
            search_container.pack(fill=tk.X, pady=(0, 8))

            tk.Label(
                search_container,
                text="البحث:",
                font=("Tajawal", 12),
                bg="#FFFFFF",
                fg="#666"
            ).pack(side=tk.RIGHT, padx=(0, 8))

            responsible_search_entry = tk.Entry(
                search_container,
                font=("Tajawal", 12),
                bd=1,
                relief=tk.SOLID,
                highlightthickness=1,
                highlightcolor="#1E3A5F"
            )
            responsible_search_entry.pack(side=tk.RIGHT, fill=tk.X, expand=True)

            # قائمة البحث
            list_frame = tk.Frame(left_resp, bg="#FFFFFF", relief=tk.RIDGE, bd=1)
            list_frame.pack(fill=tk.BOTH, expand=True)

            scrollbar = tk.Scrollbar(list_frame, width=12)
            scrollbar.pack(side=tk.LEFT, fill=tk.Y)

            responsible_listbox = tk.Listbox(
                list_frame,
                font=("Tajawal", 11),
                height=4,
                bd=0,
                highlightthickness=0,
                yscrollcommand=scrollbar.set,
                bg="#FAFAFA",
                fg="#333333",
                selectbackground="#E3F2FD",
                selectforeground="#0D47A1"
            )
            responsible_listbox.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)
            scrollbar.config(command=responsible_listbox.yview)

            # زر التعيين
            assign_resp_btn = tk.Button(
                left_resp,
                text="تعيين كرئيس",
                font=("Tajawal", 11, "bold"),
                bg="#3498DB",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=15,
                pady=6
            )
            assign_resp_btn.pack(pady=(8, 0))

            # القسم الثاني: أعضاء المسار - مقسم إلى قسمين
            members_container = tk.Frame(main_container, bg="#F5F5F5")
            members_container.pack(fill=tk.BOTH, expand=True)

            # القسم الأيمن - منسوبي المدينة
            city_staff_frame = tk.Frame(members_container, bg="#FFFFFF", bd=1, relief=tk.RIDGE)
            city_staff_frame.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True, padx=(5, 0))

            # رأس قسم منسوبي المدينة
            city_header = tk.Frame(city_staff_frame, bg="#1B4F72", height=40)
            city_header.pack(fill=tk.X)
            city_header.pack_propagate(False)

            city_header_content = tk.Frame(city_header, bg="#1B4F72")
            city_header_content.pack(expand=True)

            tk.Label(
                city_header_content,
                text="أعضاء المسار - منسوبي المدينة",
                font=("Tajawal", 14, "bold"),
                bg="#1B4F72",
                fg="white"
            ).pack(side=tk.RIGHT)

            city_count_label = tk.Label(
                city_header_content,
                text="",
                font=("Tajawal", 12),
                bg="#1B4F72",
                fg="white"
            )
            city_count_label.pack(side=tk.RIGHT, padx=(15, 0))

            # محتوى قسم منسوبي المدينة
            city_content = tk.Frame(city_staff_frame, bg="#FFFFFF", padx=15, pady=15)
            city_content.pack(fill=tk.BOTH, expand=True)

            # البحث في منسوبي المدينة
            city_search_frame = tk.Frame(city_content, bg="#FFFFFF")
            city_search_frame.pack(fill=tk.X, pady=(0, 10))

            tk.Label(
                city_search_frame,
                text="البحث:",
                font=("Tajawal", 12),
                bg="#FFFFFF",
                fg="#666"
            ).pack(side=tk.RIGHT, padx=(0, 8))

            city_search_entry = tk.Entry(
                city_search_frame,
                font=("Tajawal", 12),
                bd=1,
                relief=tk.SOLID,
                highlightthickness=1,
                highlightcolor="#1B4F72"
            )
            city_search_entry.pack(side=tk.RIGHT, fill=tk.X, expand=True)

            # قائمة البحث لمنسوبي المدينة
            city_search_list_frame = tk.Frame(city_content, bg="#FFFFFF", relief=tk.RIDGE, bd=1)
            city_search_list_frame.pack(fill=tk.BOTH, expand=True, pady=(0, 10))

            city_search_scrollbar = tk.Scrollbar(city_search_list_frame, width=12)
            city_search_scrollbar.pack(side=tk.LEFT, fill=tk.Y)

            city_search_listbox = tk.Listbox(
                city_search_list_frame,
                font=("Tajawal", 11),
                height=6,
                bd=0,
                highlightthickness=0,
                yscrollcommand=city_search_scrollbar.set,
                bg="#FAFAFA",
                fg="#333333",
                selectbackground="#D4E6F1",
                selectforeground="#1B4F72"
            )
            city_search_listbox.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)
            city_search_scrollbar.config(command=city_search_listbox.yview)

            # إطار الأزرار لمنسوبي المدينة
            city_buttons_frame = tk.Frame(city_content, bg="#FFFFFF")
            city_buttons_frame.pack(pady=(5, 10))

            # زر إضافة منسوب مدينة
            add_city_btn = tk.Button(
                city_buttons_frame,
                text="إضافة العضو",
                font=("Tajawal", 11, "bold"),
                bg="#2874A6",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=15,
                pady=6
            )
            add_city_btn.pack(side=tk.RIGHT, padx=10)

            # زر حذف منسوب مدينة
            remove_city_btn = tk.Button(
                city_buttons_frame,
                text="حذف العضو المحدد",
                font=("Tajawal", 11, "bold"),
                bg="#B03A2E",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=15,
                pady=6
            )
            remove_city_btn.pack(side=tk.RIGHT, padx=10)

            # خط فاصل
            tk.Frame(city_content, bg="#E0E0E0", height=2).pack(fill=tk.X, pady=10)

            # الأعضاء الحاليون من منسوبي المدينة
            tk.Label(
                city_content,
                text="الأعضاء الحاليون:",
                font=("Tajawal", 12, "bold"),
                bg="#FFFFFF",
                fg="#1B4F72"
            ).pack(anchor=tk.E, pady=(0, 10))

            city_members_frame = tk.Frame(city_content, bg="#FFFFFF", relief=tk.RIDGE, bd=1)
            city_members_frame.pack(fill=tk.BOTH, expand=True)

            city_members_scrollbar = tk.Scrollbar(city_members_frame, width=12)
            city_members_scrollbar.pack(side=tk.LEFT, fill=tk.Y)

            city_members_listbox = tk.Listbox(
                city_members_frame,
                font=("Tajawal", 11),
                height=6,
                bd=0,
                highlightthickness=0,
                yscrollcommand=city_members_scrollbar.set,
                bg="#EBF5FB",
                fg="#333333",
                selectbackground="#FFEBEE",
                selectforeground="#B71C1C"
            )
            city_members_listbox.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)
            city_members_scrollbar.config(command=city_members_listbox.yview)

            # القسم الأيسر - المتعاونين
            collaborators_frame = tk.Frame(members_container, bg="#FFFFFF", bd=1, relief=tk.RIDGE)
            collaborators_frame.pack(side=tk.LEFT, fill=tk.BOTH, expand=True, padx=(0, 5))

            # رأس قسم المتعاونين
            collab_header = tk.Frame(collaborators_frame, bg="#424949", height=40)
            collab_header.pack(fill=tk.X)
            collab_header.pack_propagate(False)

            collab_header_content = tk.Frame(collab_header, bg="#424949")
            collab_header_content.pack(expand=True)

            tk.Label(
                collab_header_content,
                text="أعضاء المسار - المتعاونين",
                font=("Tajawal", 14, "bold"),
                bg="#424949",
                fg="white"
            ).pack(side=tk.RIGHT)

            collab_count_label = tk.Label(
                collab_header_content,
                text="",
                font=("Tajawal", 12),
                bg="#424949",
                fg="white"
            )
            collab_count_label.pack(side=tk.RIGHT, padx=(15, 0))

            # محتوى قسم المتعاونين
            collab_content = tk.Frame(collaborators_frame, bg="#FFFFFF", padx=15, pady=15)
            collab_content.pack(fill=tk.BOTH, expand=True)

            # البحث في المتعاونين
            collab_search_frame = tk.Frame(collab_content, bg="#FFFFFF")
            collab_search_frame.pack(fill=tk.X, pady=(0, 10))

            tk.Label(
                collab_search_frame,
                text="البحث:",
                font=("Tajawal", 12),
                bg="#FFFFFF",
                fg="#666"
            ).pack(side=tk.RIGHT, padx=(0, 8))

            collab_search_entry = tk.Entry(
                collab_search_frame,
                font=("Tajawal", 12),
                bd=1,
                relief=tk.SOLID,
                highlightthickness=1,
                highlightcolor="#424949"
            )
            collab_search_entry.pack(side=tk.RIGHT, fill=tk.X, expand=True)

            # قائمة البحث للمتعاونين
            collab_search_list_frame = tk.Frame(collab_content, bg="#FFFFFF", relief=tk.RIDGE, bd=1)
            collab_search_list_frame.pack(fill=tk.BOTH, expand=True, pady=(0, 10))

            collab_search_scrollbar = tk.Scrollbar(collab_search_list_frame, width=12)
            collab_search_scrollbar.pack(side=tk.LEFT, fill=tk.Y)

            collab_search_listbox = tk.Listbox(
                collab_search_list_frame,
                font=("Tajawal", 11),
                height=6,
                bd=0,
                highlightthickness=0,
                yscrollcommand=collab_search_scrollbar.set,
                bg="#FAFAFA",
                fg="#333333",
                selectbackground="#D5DBDB",
                selectforeground="#212F3C"
            )
            collab_search_listbox.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)
            collab_search_scrollbar.config(command=collab_search_listbox.yview)

            # إطار الأزرار للمتعاونين
            collab_buttons_frame = tk.Frame(collab_content, bg="#FFFFFF")
            collab_buttons_frame.pack(pady=(5, 10))

            # زر إضافة متعاون
            add_collab_btn = tk.Button(
                collab_buttons_frame,
                text="إضافة العضو",
                font=("Tajawal", 11, "bold"),
                bg="#5D6D7E",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=15,
                pady=6
            )
            add_collab_btn.pack(side=tk.RIGHT, padx=10)

            # زر حذف متعاون
            remove_collab_btn = tk.Button(
                collab_buttons_frame,
                text="حذف العضو المحدد",
                font=("Tajawal", 11, "bold"),
                bg="#B03A2E",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=15,
                pady=6
            )
            remove_collab_btn.pack(side=tk.RIGHT, padx=10)

            # خط فاصل
            tk.Frame(collab_content, bg="#E0E0E0", height=2).pack(fill=tk.X, pady=10)

            # الأعضاء الحاليون من المتعاونين
            tk.Label(
                collab_content,
                text="الأعضاء الحاليون:",
                font=("Tajawal", 12, "bold"),
                bg="#FFFFFF",
                fg="#424949"
            ).pack(anchor=tk.E, pady=(0, 10))

            collab_members_frame = tk.Frame(collab_content, bg="#FFFFFF", relief=tk.RIDGE, bd=1)
            collab_members_frame.pack(fill=tk.BOTH, expand=True)

            collab_members_scrollbar = tk.Scrollbar(collab_members_frame, width=12)
            collab_members_scrollbar.pack(side=tk.LEFT, fill=tk.Y)

            collab_members_listbox = tk.Listbox(
                collab_members_frame,
                font=("Tajawal", 11),
                height=6,
                bd=0,
                highlightthickness=0,
                yscrollcommand=collab_members_scrollbar.set,
                bg="#F4F6F6",
                fg="#333333",
                selectbackground="#FFEBEE",
                selectforeground="#B71C1C"
            )
            collab_members_listbox.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)
            collab_members_scrollbar.config(command=collab_members_listbox.yview)

            # الشريط السفلي
            footer_frame = tk.Frame(manage_window, bg="#1E3A5F", height=50)
            footer_frame.pack(fill=tk.X)
            footer_frame.pack_propagate(False)

            close_btn = tk.Button(
                footer_frame,
                text="إغلاق",
                font=("Tajawal", 13, "bold"),
                bg="#34495E",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=35,
                pady=10,
                command=lambda: [manage_window.destroy(), search_subjects()]
            )
            close_btn.place(relx=0.5, rely=0.5, anchor=tk.CENTER)

            # maps لتخزين معرفات المدرسين
            manage_window.city_members_map = {}
            manage_window.collab_members_map = {}
            manage_window.city_search_map = {}
            manage_window.collab_search_map = {}
            manage_window.responsible_map = {}

            # تحميل البيانات الحالية
            def load_current_data():
                cursor = self.db_conn.cursor()

                # تحميل رئيس المسار
                cursor.execute("""
                    SELECT t.id, t.name, t.rank, t.workplace
                    FROM faculty_subject_paths fsp
                    JOIN teachers t ON fsp.teacher_id = t.id
                    WHERE fsp.subject_name = ? AND fsp.is_head = 1
                """, (subject_name,))

                responsible = cursor.fetchone()
                if responsible:
                    current_responsible_label.config(
                        text=f"{responsible[2]} - {responsible[1]}",
                        font=("Tajawal", 11, "bold")
                    )
                    remove_resp_btn.config(state=tk.NORMAL)
                else:
                    current_responsible_label.config(
                        text="لم يتم تعيين رئيس بعد",
                        fg="#999999",
                        font=("Tajawal", 11, "italic")
                    )
                    remove_resp_btn.config(state=tk.DISABLED)

                # تحميل أعضاء المسار - منسوبي المدينة
                city_members_listbox.delete(0, tk.END)
                manage_window.city_members_map.clear()

                cursor.execute("""
                    SELECT t.id, t.name, t.rank, t.workplace
                    FROM faculty_subject_paths fsp
                    JOIN teachers t ON fsp.teacher_id = t.id
                    WHERE fsp.subject_name = ? AND fsp.is_head = 0
                    AND t.category = 'منسوبي المدينة'
                    ORDER BY t.name
                """, (subject_name,))

                city_members = cursor.fetchall()

                # الحصول على المسارات الأخرى لكل مدرس
                for i, (teacher_id, name, rank, workplace) in enumerate(city_members):
                    # استعلام منفصل للحصول على المسارات الأخرى
                    cursor.execute("""
                        SELECT subject_name 
                        FROM faculty_subject_paths 
                        WHERE teacher_id = ? AND subject_name != ?
                    """, (teacher_id, subject_name))

                    other_paths = [row[0] for row in cursor.fetchall()]

                    display_text = f"{rank} - {name}"
                    if other_paths:
                        display_text += f" (عضو أيضاً في: {', '.join(other_paths)})"

                    city_members_listbox.insert(tk.END, display_text)
                    manage_window.city_members_map[i] = teacher_id

                city_count_label.config(text=f"العدد: {len(city_members)}")

                # تحميل أعضاء المسار - المتعاونين
                collab_members_listbox.delete(0, tk.END)
                manage_window.collab_members_map.clear()

                cursor.execute("""
                    SELECT t.id, t.name, t.rank, t.workplace, t.category
                    FROM faculty_subject_paths fsp
                    JOIN teachers t ON fsp.teacher_id = t.id
                    WHERE fsp.subject_name = ? AND fsp.is_head = 0
                    AND t.category IN ('متعاون', 'متعاون مدني', 'متعاون عسكري', 'متعاون عسكري متقاعد')
                    ORDER BY t.name
                """, (subject_name,))

                collab_members = cursor.fetchall()

                # الحصول على المسارات الأخرى لكل متعاون
                for i, (teacher_id, name, rank, workplace, category) in enumerate(collab_members):
                    # استعلام منفصل للحصول على المسارات الأخرى
                    cursor.execute("""
                        SELECT subject_name 
                        FROM faculty_subject_paths 
                        WHERE teacher_id = ? AND subject_name != ?
                    """, (teacher_id, subject_name))

                    other_paths = [row[0] for row in cursor.fetchall()]

                    display_text = f"{rank} - {name} ({category})"
                    if other_paths:
                        display_text += f" (عضو أيضاً في: {', '.join(other_paths)})"

                    collab_members_listbox.insert(tk.END, display_text)
                    manage_window.collab_members_map[i] = teacher_id

                collab_count_label.config(text=f"العدد: {len(collab_members)}")

            # البحث عن المدرسين لرئيس المسار (يبحث في الجميع)
            def search_responsible_teachers(*args):
                search_term = responsible_search_entry.get().strip()
                responsible_listbox.delete(0, tk.END)

                if len(search_term) < 1:
                    return

                cursor = self.db_conn.cursor()
                cursor.execute("""
                    SELECT t.id, t.name, t.rank, t.workplace, t.category
                    FROM teachers t
                    WHERE LOWER(t.name) LIKE LOWER(?)
                    ORDER BY t.name
                    LIMIT 20
                """, (f'%{search_term}%',))

                teachers = cursor.fetchall()
                manage_window.responsible_map.clear()

                for i, (teacher_id, name, rank, workplace, category) in enumerate(teachers):
                    # الحصول على المسارات لكل مدرس
                    cursor.execute("""
                        SELECT subject_name 
                        FROM faculty_subject_paths 
                        WHERE teacher_id = ?
                    """, (teacher_id,))

                    paths = [row[0] for row in cursor.fetchall()]

                    display_text = f"{rank} - {name}"
                    if paths:
                        display_text += f" (عضو في: {', '.join(paths)})"

                    responsible_listbox.insert(tk.END, display_text)
                    manage_window.responsible_map[i] = teacher_id

            # البحث في منسوبي المدينة
            def search_city_teachers(*args):
                search_term = city_search_entry.get().strip()
                city_search_listbox.delete(0, tk.END)

                if len(search_term) < 1:
                    return

                cursor = self.db_conn.cursor()
                cursor.execute("""
                    SELECT t.id, t.name, t.rank, t.workplace
                    FROM teachers t
                    WHERE LOWER(t.name) LIKE LOWER(?)
                    AND t.category = 'منسوبي المدينة'
                    ORDER BY t.name
                    LIMIT 20
                """, (f'%{search_term}%',))

                teachers = cursor.fetchall()
                manage_window.city_search_map.clear()

                for i, (teacher_id, name, rank, workplace) in enumerate(teachers):
                    # الحصول على المسارات لكل مدرس
                    cursor.execute("""
                        SELECT subject_name 
                        FROM faculty_subject_paths 
                        WHERE teacher_id = ?
                    """, (teacher_id,))

                    paths = [row[0] for row in cursor.fetchall()]

                    display_text = f"{rank} - {name}"
                    if paths:
                        display_text += f" (عضو في: {', '.join(paths)})"

                    city_search_listbox.insert(tk.END, display_text)
                    manage_window.city_search_map[i] = teacher_id

            # البحث في المتعاونين
            def search_collab_teachers(*args):
                search_term = collab_search_entry.get().strip()
                collab_search_listbox.delete(0, tk.END)

                if len(search_term) < 1:
                    return

                cursor = self.db_conn.cursor()
                cursor.execute("""
                    SELECT t.id, t.name, t.rank, t.workplace, t.category
                    FROM teachers t
                    WHERE LOWER(t.name) LIKE LOWER(?)
                    AND t.category IN ('متعاون', 'متعاون مدني', 'متعاون عسكري', 'متعاون عسكري متقاعد')
                    ORDER BY t.name
                    LIMIT 20
                """, (f'%{search_term}%',))

                teachers = cursor.fetchall()
                manage_window.collab_search_map.clear()

                for i, (teacher_id, name, rank, workplace, category) in enumerate(teachers):
                    # الحصول على المسارات لكل مدرس
                    cursor.execute("""
                        SELECT subject_name 
                        FROM faculty_subject_paths 
                        WHERE teacher_id = ?
                    """, (teacher_id,))

                    paths = [row[0] for row in cursor.fetchall()]

                    display_text = f"{rank} - {name} ({category})"
                    if paths:
                        display_text += f" (عضو في: {', '.join(paths)})"

                    collab_search_listbox.insert(tk.END, display_text)
                    manage_window.collab_search_map[i] = teacher_id

            # تعيين رئيس مسار - السماح بالتكرار
            def assign_responsible():
                selection = responsible_listbox.curselection()
                if not selection:
                    messagebox.showwarning("تنبيه", "يرجى اختيار مدرس من القائمة")
                    return

                teacher_id = manage_window.responsible_map[selection[0]]

                # التحقق من عدم وجود المدرس كرئيس في نفس المسار
                cursor = self.db_conn.cursor()
                cursor.execute("""
                    SELECT COUNT(*) FROM faculty_subject_paths
                    WHERE teacher_id = ? AND subject_name = ? AND is_head = 1
                """, (teacher_id, subject_name))

                if cursor.fetchone()[0] > 0:
                    messagebox.showwarning("تنبيه", "هذا المدرس هو الرئيس الحالي بالفعل")
                    return

                if messagebox.askyesno("تأكيد", "هل تريد تعيين هذا المدرس كرئيس للمسار؟"):
                    try:
                        cursor = self.db_conn.cursor()

                        # إلغاء أي رئيس سابق
                        cursor.execute("""
                            UPDATE faculty_subject_paths 
                            SET is_head = 0 
                            WHERE subject_name = ? AND is_head = 1
                        """, (subject_name,))

                        # التحقق من وجود السجل أولاً
                        cursor.execute("""
                            SELECT COUNT(*) FROM faculty_subject_paths
                            WHERE subject_name = ? AND teacher_id = ?
                        """, (subject_name, teacher_id))

                        if cursor.fetchone()[0] > 0:
                            # تحديث السجل الموجود
                            cursor.execute("""
                                UPDATE faculty_subject_paths 
                                SET is_head = 1 
                                WHERE subject_name = ? AND teacher_id = ?
                            """, (subject_name, teacher_id))
                        else:
                            # إضافة سجل جديد
                            cursor.execute("""
                                INSERT INTO faculty_subject_paths (subject_name, teacher_id, is_head, created_date)
                                VALUES (?, ?, 1, ?)
                            """, (subject_name, teacher_id, datetime.now().strftime("%Y-%m-%d %H:%M:%S")))

                        self.db_conn.commit()

                        load_current_data()
                        responsible_search_entry.delete(0, tk.END)
                        responsible_listbox.delete(0, tk.END)

                        messagebox.showinfo("نجاح", "تم تعيين رئيس المسار بنجاح")

                    except Exception as e:
                        messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

            # حذف رئيس المسار
            def remove_responsible():
                if messagebox.askyesno("تأكيد", "هل تريد حذف الرئيس الحالي؟"):
                    try:
                        cursor = self.db_conn.cursor()
                        cursor.execute("""
                            DELETE FROM faculty_subject_paths 
                            WHERE subject_name = ? AND is_head = 1
                        """, (subject_name,))

                        self.db_conn.commit()
                        load_current_data()
                        messagebox.showinfo("نجاح", "تم حذف رئيس المسار بنجاح")

                    except Exception as e:
                        messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

            # إضافة منسوب مدينة - السماح بالتكرار
            def add_city_member():
                selection = city_search_listbox.curselection()
                if not selection:
                    messagebox.showwarning("تنبيه", "يرجى اختيار مدرس من القائمة")
                    return

                teacher_id = manage_window.city_search_map[selection[0]]

                # التحقق من عدم وجود المدرس في نفس المسار
                cursor = self.db_conn.cursor()
                cursor.execute("""
                    SELECT COUNT(*) FROM faculty_subject_paths 
                    WHERE teacher_id = ? AND subject_name = ?
                """, (teacher_id, subject_name))

                if cursor.fetchone()[0] > 0:
                    messagebox.showwarning("تنبيه", "هذا المدرس موجود بالفعل في هذا المسار")
                    return

                try:
                    cursor = self.db_conn.cursor()

                    cursor.execute("""
                        INSERT INTO faculty_subject_paths (subject_name, teacher_id, is_head, created_date)
                        VALUES (?, ?, 0, ?)
                    """, (subject_name, teacher_id, datetime.now().strftime("%Y-%m-%d %H:%M:%S")))

                    self.db_conn.commit()

                    load_current_data()
                    city_search_entry.delete(0, tk.END)
                    city_search_listbox.delete(0, tk.END)

                    messagebox.showinfo("نجاح", "تمت إضافة العضو بنجاح")

                except Exception as e:
                    messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

            # إضافة متعاون - السماح بالتكرار
            def add_collab_member():
                selection = collab_search_listbox.curselection()
                if not selection:
                    messagebox.showwarning("تنبيه", "يرجى اختيار مدرس من القائمة")
                    return

                teacher_id = manage_window.collab_search_map[selection[0]]

                # التحقق من عدم وجود المدرس في نفس المسار
                cursor = self.db_conn.cursor()
                cursor.execute("""
                    SELECT COUNT(*) FROM faculty_subject_paths 
                    WHERE teacher_id = ? AND subject_name = ?
                """, (teacher_id, subject_name))

                if cursor.fetchone()[0] > 0:
                    messagebox.showwarning("تنبيه", "هذا المدرس موجود بالفعل في هذا المسار")
                    return

                try:
                    cursor = self.db_conn.cursor()

                    cursor.execute("""
                        INSERT INTO faculty_subject_paths (subject_name, teacher_id, is_head, created_date)
                        VALUES (?, ?, 0, ?)
                    """, (subject_name, teacher_id, datetime.now().strftime("%Y-%m-%d %H:%M:%S")))

                    self.db_conn.commit()

                    load_current_data()
                    collab_search_entry.delete(0, tk.END)
                    collab_search_listbox.delete(0, tk.END)

                    messagebox.showinfo("نجاح", "تمت إضافة العضو بنجاح")

                except Exception as e:
                    messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

            # حذف منسوب مدينة
            def remove_city_member():
                selection = city_members_listbox.curselection()
                if not selection:
                    messagebox.showwarning("تنبيه", "يرجى اختيار عضو للحذف")
                    return

                teacher_id = manage_window.city_members_map[selection[0]]

                if messagebox.askyesno("تأكيد", "هل تريد حذف هذا العضو من المسار؟"):
                    try:
                        cursor = self.db_conn.cursor()
                        cursor.execute("""
                            DELETE FROM faculty_subject_paths 
                            WHERE subject_name = ? AND teacher_id = ? AND is_head = 0
                        """, (subject_name, teacher_id))

                        self.db_conn.commit()
                        load_current_data()

                    except Exception as e:
                        messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

            # حذف متعاون
            def remove_collab_member():
                selection = collab_members_listbox.curselection()
                if not selection:
                    messagebox.showwarning("تنبيه", "يرجى اختيار عضو للحذف")
                    return

                teacher_id = manage_window.collab_members_map[selection[0]]

                if messagebox.askyesno("تأكيد", "هل تريد حذف هذا العضو من المسار؟"):
                    try:
                        cursor = self.db_conn.cursor()
                        cursor.execute("""
                            DELETE FROM faculty_subject_paths 
                            WHERE subject_name = ? AND teacher_id = ? AND is_head = 0
                        """, (subject_name, teacher_id))

                        self.db_conn.commit()
                        load_current_data()

                    except Exception as e:
                        messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

            # ربط الأحداث
            responsible_search_entry.bind('<KeyRelease>', search_responsible_teachers)
            city_search_entry.bind('<KeyRelease>', search_city_teachers)
            collab_search_entry.bind('<KeyRelease>', search_collab_teachers)

            assign_resp_btn.config(command=assign_responsible)
            remove_resp_btn.config(command=remove_responsible)
            add_city_btn.config(command=add_city_member)
            add_collab_btn.config(command=add_collab_member)
            remove_city_btn.config(command=remove_city_member)
            remove_collab_btn.config(command=remove_collab_member)

            # تحميل البيانات الأولية
            load_current_data()

        # دالة لحذف المسار
        def delete_path(subject_name):
            """حذف المسار"""
            if messagebox.askyesno("تأكيد الحذف", f"هل تريد حذف مسار {subject_name}؟\nسيتم حذف جميع التعيينات"):
                cursor = self.db_conn.cursor()
                cursor.execute("DELETE FROM faculty_subject_paths WHERE subject_name = ?", (subject_name,))
                self.db_conn.commit()
                messagebox.showinfo("نجاح", "تم حذف المسار بنجاح")
                # تحديث العرض
                faculty_window.destroy()
                self._open_faculty_paths_page()

        # دالة لإنشاء بطاقة مسار محدثة
        def create_path_card(parent, subject_name, row, col):
            """إنشاء بطاقة مسار بتصميم محسن وحجم ملائم"""
            # إطار البطاقة الرئيسي - حجم ملائم
            card_frame = tk.Frame(parent, bg="#FFFFFF", relief=tk.RAISED, bd=2)
            card_frame.grid(row=row, column=col, padx=15, pady=15, sticky="nsew")

            # شريط علوي بلون موحد
            border_color = "#1E3A5F"  # لون موحد لجميع المسارات
            icon = "📚"

            # شريط علوي ملون
            top_bar = tk.Frame(card_frame, bg=border_color, height=8)
            top_bar.pack(fill=tk.X, side=tk.TOP)

            # محتوى البطاقة
            card_content = tk.Frame(card_frame, bg="#FFFFFF", padx=20, pady=20)
            card_content.pack(fill=tk.BOTH, expand=True)

            # إطار العنوان
            title_frame = tk.Frame(card_content, bg="#FFFFFF")
            title_frame.pack(fill=tk.X, pady=(0, 20))

            # أيقونة
            tk.Label(
                title_frame,
                text=icon,
                font=("Arial", 36),
                bg="#FFFFFF"
            ).pack(side=tk.TOP, pady=(0, 10))

            # اسم المسار
            tk.Label(
                title_frame,
                text=subject_name,
                font=("Tajawal", 18, "bold"),
                bg="#FFFFFF",
                fg="#1E3A5F",
                wraplength=250,
                justify=tk.CENTER
            ).pack(side=tk.TOP)

            # خط فاصل
            separator1 = tk.Frame(card_content, bg="#E0E0E0", height=2)
            separator1.pack(fill=tk.X, pady=15)

            # الحصول على البيانات من قاعدة البيانات
            cursor = self.db_conn.cursor()

            # عدد رئيس المسار
            cursor.execute("""
                SELECT COUNT(*) FROM faculty_subject_paths 
                WHERE subject_name = ? AND is_head = 1
            """, (subject_name,))
            head_count = cursor.fetchone()[0]

            # عدد منسوبي المدينة
            cursor.execute("""
                SELECT COUNT(*) FROM faculty_subject_paths fsp
                JOIN teachers t ON fsp.teacher_id = t.id
                WHERE fsp.subject_name = ? AND fsp.is_head = 0
                AND t.category = 'منسوبي المدينة'
            """, (subject_name,))
            city_count = cursor.fetchone()[0]

            # عدد المتعاونين
            cursor.execute("""
                SELECT COUNT(*) FROM faculty_subject_paths fsp
                JOIN teachers t ON fsp.teacher_id = t.id
                WHERE fsp.subject_name = ? AND fsp.is_head = 0
                AND t.category IN ('متعاون', 'متعاون مدني', 'متعاون عسكري', 'متعاون عسكري متقاعد')
            """, (subject_name,))
            collab_count = cursor.fetchone()[0]

            # قسم رئيس المسار
            head_frame = tk.Frame(card_content, bg="#FFF8DC", relief=tk.GROOVE, bd=1)
            head_frame.pack(fill=tk.X, pady=(0, 10))

            head_inner = tk.Frame(head_frame, bg="#FFF8DC", padx=10, pady=10)
            head_inner.pack(fill=tk.X)

            head_text = f"👑 رئيس المسار ({head_count})" if head_count > 0 else "👑 رئيس المسار (غير محدد)"
            tk.Label(
                head_inner,
                text=head_text,
                font=("Tajawal", 13, "bold"),
                bg="#FFF8DC",
                fg="#8B4513" if head_count > 0 else "#999999",
                cursor="hand2"
            ).pack(anchor=tk.W)

            # خط فاصل
            separator2 = tk.Frame(card_content, bg="#E0E0E0", height=1)
            separator2.pack(fill=tk.X, pady=5)

            # قسم منسوبي المدينة
            city_frame = tk.Frame(card_content, bg="#E8F5E9", relief=tk.GROOVE, bd=1)
            city_frame.pack(fill=tk.X, pady=(0, 10))

            city_inner = tk.Frame(city_frame, bg="#E8F5E9", padx=10, pady=10)
            city_inner.pack(fill=tk.X)

            city_label = tk.Label(
                city_inner,
                text=f"🏢 منسوبي المدينة ({city_count})",
                font=("Tajawal", 13, "bold"),
                bg="#E8F5E9",
                fg="#1B5E20",
                cursor="hand2"
            )
            city_label.pack(anchor=tk.W)

            # ربط النقر على منسوبي المدينة
            city_label.bind("<Button-1>", lambda e: show_members(subject_name, "منسوبي المدينة"))
            city_label.bind("<Enter>",
                            lambda e: city_label.config(fg="#2E7D32", font=("Tajawal", 13, "bold", "underline")))
            city_label.bind("<Leave>", lambda e: city_label.config(fg="#1B5E20", font=("Tajawal", 13, "bold")))

            # خط فاصل
            separator3 = tk.Frame(card_content, bg="#E0E0E0", height=1)
            separator3.pack(fill=tk.X, pady=5)

            # قسم المتعاونين
            collab_frame = tk.Frame(card_content, bg="#E3F2FD", relief=tk.GROOVE, bd=1)
            collab_frame.pack(fill=tk.X)

            collab_inner = tk.Frame(collab_frame, bg="#E3F2FD", padx=10, pady=10)
            collab_inner.pack(fill=tk.X)

            collab_label = tk.Label(
                collab_inner,
                text=f"🤝 المتعاونين ({collab_count})",
                font=("Tajawal", 13, "bold"),
                bg="#E3F2FD",
                fg="#0D47A1",
                cursor="hand2"
            )
            collab_label.pack(anchor=tk.W)

            # ربط النقر على المتعاونين
            collab_label.bind("<Button-1>", lambda e: show_members(subject_name, "المتعاونين"))
            collab_label.bind("<Enter>",
                              lambda e: collab_label.config(fg="#1976D2", font=("Tajawal", 13, "bold", "underline")))
            collab_label.bind("<Leave>", lambda e: collab_label.config(fg="#0D47A1", font=("Tajawal", 13, "bold")))

            # مساحة فارغة
            tk.Frame(card_content, bg="#FFFFFF", height=20).pack()

            # أزرار الإدارة في الأسفل
            buttons_frame = tk.Frame(card_content, bg="#FFFFFF")
            buttons_frame.pack(side=tk.BOTTOM, fill=tk.X, pady=(10, 0))

            # زر إدارة المسار
            manage_btn = tk.Button(
                buttons_frame,
                text="إدارة المسار",
                font=("Tajawal", 12, "bold"),
                bg="#37474F",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=20,
                pady=8,
                command=lambda: manage_path(subject_name)
            )
            manage_btn.pack(side=tk.LEFT, fill=tk.X, expand=True, padx=(0, 5))

            # زر حذف المسار
            delete_btn = tk.Button(
                buttons_frame,
                text="حذف المسار",
                font=("Tajawal", 12, "bold"),
                bg="#5D4037",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=20,
                pady=8,
                command=lambda: delete_path(subject_name)
            )
            delete_btn.pack(side=tk.RIGHT, fill=tk.X, expand=True, padx=(5, 0))

            # تأثيرات hover للأزرار
            manage_btn.bind("<Enter>", lambda e: manage_btn.config(bg="#455A64"))
            manage_btn.bind("<Leave>", lambda e: manage_btn.config(bg="#37474F"))

            delete_btn.bind("<Enter>", lambda e: delete_btn.config(bg="#6D4C41"))
            delete_btn.bind("<Leave>", lambda e: delete_btn.config(bg="#5D4037"))

            # تأثير hover للبطاقة كاملة
            def on_card_enter(e):
                card_frame.config(bd=4)

            def on_card_leave(e):
                card_frame.config(bd=2)

            card_frame.bind("<Enter>", on_card_enter)
            card_frame.bind("<Leave>", on_card_leave)

            # حجم ثابت للبطاقة - ملائم
            card_frame.configure(width=320, height=450)
            card_frame.grid_propagate(False)

        # إنشاء جدول جديد إذا لم يكن موجوداً
        cursor = self.db_conn.cursor()
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS faculty_subject_paths (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                subject_name TEXT NOT NULL,
                teacher_id INTEGER NOT NULL,
                is_head INTEGER DEFAULT 0,
                created_date TEXT,
                FOREIGN KEY (teacher_id) REFERENCES teachers(id) ON DELETE CASCADE
            )
        """)
        self.db_conn.commit()

        # الحصول على جميع المواد
        all_subjects = []

        # المواد العامة
        cursor.execute("SELECT subject_name FROM general_subjects ORDER BY subject_name")
        general_subjects = cursor.fetchall()
        all_subjects.extend([s[0] for s in general_subjects])

        # المواد التخصصية (جميعها بدون تحديد دورة)
        cursor.execute("SELECT DISTINCT subject_name FROM specialized_subjects ORDER BY subject_name")
        specialized_subjects = cursor.fetchall()
        all_subjects.extend([s[0] for s in specialized_subjects])

        # إزالة التكرارات وترتيب القائمة
        all_subjects = list(set(all_subjects))
        all_subjects.sort()

        # عرض البطاقات في شبكة
        columns = 4  # عدد الأعمدة
        for idx, subject_name in enumerate(all_subjects):
            row = idx // columns
            col = idx % columns
            create_path_card(content_frame, subject_name, row, col)

        # تكوين أوزان الأعمدة - حجم ملائم
        for i in range(columns):
            content_frame.grid_columnconfigure(i, weight=1, minsize=350)

        # ربط البحث
        def search_subjects(event=None):
            """البحث في المواد"""
            search_text = search_entry.get().strip().lower()

            # إخفاء جميع البطاقات
            for widget in content_frame.winfo_children():
                widget.grid_forget()

            # إعادة عرض المواد المطابقة
            displayed = 0
            for idx, subject_name in enumerate(all_subjects):
                if not search_text or search_text in subject_name.lower():
                    row = displayed // columns
                    col = displayed % columns
                    create_path_card(content_frame, subject_name, row, col)
                    displayed += 1

        search_entry.bind('<KeyRelease>', search_subjects)

        # ترتيب عناصر التمرير
        canvas.pack(side="left", fill="both", expand=True)
        v_scrollbar.pack(side="right", fill="y")
        h_scrollbar.pack(side="bottom", fill="x", before=canvas)

        # شريط سفلي للمعلومات
        footer_frame = tk.Frame(faculty_window, bg="#2c3e50", height=60)
        footer_frame.pack(side=tk.BOTTOM, fill=tk.X)
        footer_frame.pack_propagate(False)

        footer_content = tk.Frame(footer_frame, bg="#2c3e50")
        footer_content.pack(expand=True)

        tk.Label(
            footer_content,
            text=f"إجمالي المسارات: {len(all_subjects)}",
            font=("Tajawal", 14, "bold"),
            bg="#2c3e50",
            fg="white"
        ).pack()

    def _export_faculty_paths_data(self):
        """تصدير بيانات مسارات هيئة التدريس إلى Excel"""
        if not EXCEL_AVAILABLE:
            messagebox.showerror("خطأ", "يجب تثبيت المكتبات المطلوبة\npip install pandas openpyxl")
            return

        try:
            from tkinter import filedialog
            import pandas as pd

            file_path = filedialog.asksaveasfilename(
                title="حفظ بيانات مسارات هيئة التدريس",
                defaultextension=".xlsx",
                initialfile=f"مسارات_هيئة_التدريس_{datetime.now().strftime('%Y%m%d_%H%M%S')}.xlsx",
                filetypes=[("Excel files", "*.xlsx"), ("All files", "*.*")]
            )

            if not file_path:
                return

            cursor = self.db_conn.cursor()

            # جمع البيانات
            data = []

            # المواد العامة
            cursor.execute("SELECT subject_name FROM general_subjects ORDER BY subject_name")
            general_subjects = cursor.fetchall()

            # المواد التخصصية
            cursor.execute("SELECT DISTINCT subject_name FROM specialized_subjects ORDER BY subject_name")
            specialized_subjects = cursor.fetchall()

            all_subjects = [(s[0], 'عامة') for s in general_subjects] + [(s[0], 'تخصصية') for s in specialized_subjects]

            for subject_name, subject_type in all_subjects:
                # رئيس المسار
                cursor.execute("""
                    SELECT t.name, t.rank FROM faculty_subject_paths fsp
                    JOIN teachers t ON fsp.teacher_id = t.id
                    WHERE fsp.subject_name = ? AND fsp.is_head = 1
                """, (subject_name,))
                head = cursor.fetchone()
                head_name = f"{head[1]} - {head[0]}" if head else "غير محدد"

                # منسوبي المدينة
                cursor.execute("""
                    SELECT t.name, t.rank FROM faculty_subject_paths fsp
                    JOIN teachers t ON fsp.teacher_id = t.id
                    WHERE fsp.subject_name = ? AND fsp.is_head = 0
                    AND t.category = 'منسوبي المدينة'
                """, (subject_name,))
                city_members = cursor.fetchall()
                city_names = [f"{m[1]} - {m[0]}" for m in city_members]

                # المتعاونين
                cursor.execute("""
                    SELECT t.name, t.rank, t.category FROM faculty_subject_paths fsp
                    JOIN teachers t ON fsp.teacher_id = t.id
                    WHERE fsp.subject_name = ? AND fsp.is_head = 0
                    AND t.category IN ('متعاون', 'متعاون مدني', 'متعاون عسكري', 'متعاون عسكري متقاعد')
                """, (subject_name,))
                collab_members = cursor.fetchall()
                collab_names = [f"{m[1]} - {m[0]} ({m[2]})" for m in collab_members]

                data.append({
                    'اسم المادة': subject_name,
                    'نوع المادة': subject_type,
                    'رئيس المسار': head_name,
                    'منسوبي المدينة': '\n'.join(city_names) if city_names else 'لا يوجد',
                    'عدد منسوبي المدينة': len(city_names),
                    'المتعاونين': '\n'.join(collab_names) if collab_names else 'لا يوجد',
                    'عدد المتعاونين': len(collab_names),
                    'العدد الكلي': 1 + len(city_names) + len(collab_names) if head else len(city_names) + len(
                        collab_names)
                })

            # إنشاء DataFrame وحفظه
            df = pd.DataFrame(data)

            with pd.ExcelWriter(file_path, engine='openpyxl') as writer:
                df.to_excel(writer, sheet_name='مسارات هيئة التدريس', index=False)

                # تنسيق
                worksheet = writer.sheets['مسارات هيئة التدريس']
                worksheet.sheet_view.rightToLeft = True

                # عرض الأعمدة
                column_widths = {
                    'A': 30,  # اسم المادة
                    'B': 15,  # نوع المادة
                    'C': 35,  # رئيس المسار
                    'D': 50,  # منسوبي المدينة
                    'E': 15,  # عدد منسوبي
                    'F': 50,  # المتعاونين
                    'G': 15,  # عدد المتعاونين
                    'H': 15  # العدد الكلي
                }

                for column, width in column_widths.items():
                    worksheet.column_dimensions[column].width = width

            messagebox.showinfo("نجاح", f"تم تصدير البيانات بنجاح\n{file_path}")

        except Exception as e:
            messagebox.showerror("خطأ", f"حدث خطأ أثناء التصدير:\n{str(e)}")

    def _add_logo_safe(self, parent_frame, logo_number):
        """إضافة شعار بصورة آمنة مع ربط الصورة بنفس الجذر الصحيح"""
        import os, sys
        from PIL import Image, ImageTk

        logo_label = tk.Label(parent_frame, bg="#E8EAF0")
        logo_label.pack(expand=True)

        # تحديد مجلد التشغيل (.py أو بعد PyInstaller)
        base_dir = sys._MEIPASS if getattr(sys, 'frozen', False) else \
            os.path.dirname(os.path.abspath(__file__))
        logos_dir = os.path.join(base_dir, "logos")

        valid_exts = ['.png', '.PNG', '.jpg', '.JPG', '.jpeg', '.JPEG', '.gif', '.GIF']
        for ext in valid_exts:
            logo_path = os.path.join(logos_dir, f"logo{logo_number}{ext}")

            if os.path.exists(logo_path):
                try:
                    pil_image = Image.open(logo_path)
                    pil_image.thumbnail((90, 90), Image.Resampling.LANCZOS)

                    # اربط الصورة بنفس الإطار الحالي
                    photo = ImageTk.PhotoImage(pil_image, master=parent_frame)

                    # احفظ المرجع حتى لا يُحذف من الذاكرة
                    if not hasattr(self, '_home_images'):
                        self._home_images = {}
                    self._home_images[f'logo{logo_number}'] = photo

                    logo_label.configure(image=photo)
                    logo_label.image = photo
                    return
                except Exception as e:
                    print(f"خطأ في معالجة {logo_path}: {e}")

        # لو لم يجد صورة صالحة
        logo_label.config(
            text=f"L{logo_number}",
            font=("Arial", 24, "bold"),
            fg="#1E3A5F"
        )

    def _on_section_hover(self, header, hover_color, enter_btn):
        """تأثير عند المرور على القسم"""
        header.config(bg=hover_color)
        enter_btn.config(fg=hover_color)

    def _on_section_leave(self, header, original_color, enter_btn):
        """إعادة الألوان الأصلية"""
        header.config(bg=original_color)
        enter_btn.config(fg=original_color)

    def _open_faculty_paths(self):
        """فتح مسارات هيئة التدريس"""
        self._show_faculty_development_message()

    def _open_programs_schedules(self):
        """فتح البرامج والجداول"""
        try:
            self.tab_control.select(2)  # الانتقال لتبويب الدورات
        except:
            messagebox.showinfo("البرامج والجداول", "سيتم فتح قسم البرامج والجداول")

    def _open_courses_management(self):
        """فتح إدارة الدورات"""
        messagebox.showinfo("إدارة الدورات", "قيد التطوير")

    def _open_hr_management(self):
        """فتح الموارد البشرية"""
        try:
            self.tab_control.select(1)  # الانتقال لتبويب المدرسين
        except:
            messagebox.showinfo("الموارد البشرية", "سيتم فتح قسم الموارد البشرية")

    #— ضفها داخل ModernTrainingDesign —
    def _logout(self):
        """تأكيد الخروج ثم إغلاق النافذة الرئيسية."""
        if messagebox.askyesno("تأكيد", "هل تريد تسجيل الخروج؟"):
            self._on_closing()
