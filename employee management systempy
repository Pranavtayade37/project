import json
import os
import re
import csv
import calendar
from datetime import datetime, date, timedelta
import socket
import tkinter as tk
from tkinter import ttk, messagebox, filedialog

# ---------------------- Storage Helpers ----------------------

DATA_DIR = "data"
EMP_FILE = os.path.join(DATA_DIR, "employees.json")
ATT_DIR = os.path.join(DATA_DIR, "attendance")
SAL_DIR = os.path.join(DATA_DIR, "salary")

DEFAULT_DATE_STR = "2025-08-19"


def ensure_dirs():
    os.makedirs(DATA_DIR, exist_ok=True)
    os.makedirs(ATT_DIR, exist_ok=True)
    os.makedirs(SAL_DIR, exist_ok=True)
    if not os.path.exists(EMP_FILE):
        with open(EMP_FILE, "w", encoding="utf-8") as f:
            json.dump([], f, indent=2)


def load_employees():
    ensure_dirs()
    try:
        with open(EMP_FILE, "r", encoding="utf-8") as f:
            return json.load(f)
    except Exception:
        return []


def save_employees(records):
    ensure_dirs()
    with open(EMP_FILE, "w", encoding="utf-8") as f:
        json.dump(records, f, indent=2)


def attendance_path(day_str):
    return os.path.join(ATT_DIR, f"{day_str}.json")


def load_attendance(day_str):
    ensure_dirs()
    path = attendance_path(day_str)
    if os.path.exists(path):
        with open(path, "r", encoding="utf-8") as f:
            return json.load(f)
    return []  # list of dicts: {emp_id, name, status, overtime, ip}


def save_attendance(day_str, rows):
    ensure_dirs()
    with open(attendance_path(day_str), "w", encoding="utf-8") as f:
        json.dump(rows, f, indent=2)


def export_csv(path, headers, rows):
    os.makedirs(os.path.dirname(path), exist_ok=True)
    with open(path, "w", newline="", encoding="utf-8") as f:
        writer = csv.DictWriter(f, fieldnames=headers)
        writer.writeheader()
        for r in rows:
            writer.writerow({h: r.get(h, "") for h in headers})
    return path

# ---------------------- Validators & Utils ----------------------

DATE_RE = re.compile(r"^\d{4}-\d{2}-\d{2}$")
MONTH_RE = re.compile(r"^\d{4}-\d{2}$")
EMAIL_RE = re.compile(r"^[^@\s]+@[^@\s]+\.[^@\s]+$")


def valid_date(s):
    if not DATE_RE.match(s):
        return False
    try:
        datetime.strptime(s, "%Y-%m-%d")
        return True
    except ValueError:
        return False


def valid_month(s):
    if not MONTH_RE.match(s):
        return False
    y, m = map(int, s.split("-"))
    return 1 <= m <= 12 and 1900 <= y <= 3000


def get_host_ip():
    try:
        return socket.gethostbyname(socket.gethostname())
    except Exception:
        return "127.0.0.1"


def month_days_iter(year, month):
    d1 = date(year, month, 1)
    _, last = calendar.monthrange(year, month)
    for i in range(last):
        yield (d1 + timedelta(days=i)).strftime("%Y-%m-%d")


def business_days_in_month(year, month):
    # Count Monday-Friday (weekday 0-4) in the given month
    cnt = 0
    for d in month_days_iter(year, month):
        dt = datetime.strptime(d, "%Y-%m-%d").date()
        if dt.weekday() < 5:
            cnt += 1
    return cnt

# ---------------------- Small Calendar Picker ----------------------

class CalendarPicker(tk.Toplevel):
    """A lightweight calendar popup that allows picking a date (YYYY-MM-DD).
    Navigation for months and years included.
    """
    def __init__(self, master, variable, year=None, month=None):
        super().__init__(master)
        self.transient(master)
        self.title("Select Date")
        self.resizable(False, False)
        self.variable = variable
        today = datetime.today()
        self.year = year or today.year
        self.month = month or today.month
        self._build()
        self.grab_set()

    def _build(self):
        hdr = ttk.Frame(self)
        hdr.pack(padx=8, pady=6)
        self.prev_year = ttk.Button(hdr, text="<<", width=3, command=self._prev_year)
        self.prev_year.grid(row=0, column=0)
        self.prev = ttk.Button(hdr, text="<", width=3, command=self._prev_month)
        self.prev.grid(row=0, column=1)
        self.title_lbl = ttk.Label(hdr, text="", width=18, anchor="center")
        self.title_lbl.grid(row=0, column=2, columnspan=3)
        self.next = ttk.Button(hdr, text=">", width=3, command=self._next_month)
        self.next.grid(row=0, column=5)
        self.next_year = ttk.Button(hdr, text=">>", width=3, command=self._next_year)
        self.next_year.grid(row=0, column=6)

        self.cal_frame = ttk.Frame(self, padding=6)
        self.cal_frame.pack()
        self._draw()

    def _draw(self):
        for w in self.cal_frame.winfo_children():
            w.destroy()
        self.title_lbl.config(text=f"{calendar.month_name[self.month]} {self.year}")
        wkdays = ["Mo","Tu","We","Th","Fr","Sa","Su"]
        for c, wd in enumerate(wkdays):
            ttk.Label(self.cal_frame, text=wd, width=4).grid(row=0, column=c)
        mc = calendar.monthcalendar(self.year, self.month)
        for r, week in enumerate(mc, start=1):
            for c, day in enumerate(week):
                if day == 0:
                    lbl = ttk.Label(self.cal_frame, text="", width=4)
                    lbl.grid(row=r, column=c)
                else:
                    btn = ttk.Button(self.cal_frame, text=str(day), width=4,
                                     command=lambda d=day: self._select(d))
                    btn.grid(row=r, column=c, padx=1, pady=1)

    def _select(self, day):
        s = f"{self.year:04d}-{self.month:02d}-{day:02d}"
        self.variable.set(s)
        self.destroy()

    def _prev_month(self):
        self.month -= 1
        if self.month < 1:
            self.month = 12
            self.year -= 1
        self._draw()

    def _next_month(self):
        self.month += 1
        if self.month > 12:
            self.month = 1
            self.year += 1
        self._draw()

    def _prev_year(self):
        self.year -= 1
        self._draw()

    def _next_year(self):
        self.year += 1
        self._draw()

# ---------------------- Login Window ----------------------

class LoginWindow(tk.Toplevel):
    def __init__(self, master, on_success):
        super().__init__(master)
        self.title("Login")
        self.geometry("360x200")
        self.resizable(False, False)
        self.protocol("WM_DELETE_WINDOW", self._on_close)

        self.on_success = on_success

        container = ttk.Frame(self, padding=16)
        container.pack(fill="both", expand=True)

        ttk.Label(container, text="Employee Management System", font=("Segoe UI", 12, "bold")).pack(pady=(0,12))

        form = ttk.Frame(container)
        form.pack(fill="x", pady=4)

        ttk.Label(form, text="Username").grid(row=0, column=0, sticky="w", padx=(0,8), pady=4)
        self.username = ttk.Entry(form)
        self.username.grid(row=0, column=1, sticky="ew", pady=4)

        ttk.Label(form, text="Password").grid(row=1, column=0, sticky="w", padx=(0,8), pady=4)
        self.password = ttk.Entry(form, show="*")
        self.password.grid(row=1, column=1, sticky="ew", pady=4)

        form.columnconfigure(1, weight=1)

        btn = ttk.Button(container, text="Login", command=self.try_login)
        btn.pack(pady=8)

        self.username.focus_set()

    def try_login(self):
        u = self.username.get().strip()
        p = self.password.get().strip()
        if u == "Admin" and p == "admin123":
            self.on_success()
            self.destroy()
        else:
            messagebox.showerror("Login Failed", "Invalid username or password.")

    def _on_close(self):
        self.master.destroy()

# ---------------------- Employees Tab ----------------------

class EmployeesTab(ttk.Frame):
    COLS = ("Emp Id", "Name", "Department", "Role", "Base Salary", "Join Date", "Phone", "Email")

    def __init__(self, master):
        super().__init__(master, padding=10)
        self.records = load_employees()
        self._build_ui()
        self._refresh_tree(self.records)

    def _build_ui(self):
        # Header & search
        header = ttk.Frame(self)
        header.pack(fill="x", pady=(0,8))
        ttk.Label(header, text="Employee Management System (No SQL . JSON Storage)", font=("Segoe UI", 11, "bold")).pack(side="left")
        # Search
        search_box = ttk.Frame(self)
        search_box.pack(fill="x", pady=(0,8))
        ttk.Label(search_box, text="Search").pack(side="left", padx=(0,6))
        self.search_var = tk.StringVar()
        ttk.Entry(search_box, textvariable=self.search_var, width=30).pack(side="left")
        ttk.Button(search_box, text="Go", command=self.on_search).pack(side="left", padx=6)
        ttk.Button(search_box, text="Export", command=self.on_export).pack(side="left", padx=6)
        ttk.Button(search_box, text="Delete Selected", command=self.on_delete_selected).pack(side="left", padx=6)

        # Form
        form = ttk.LabelFrame(self, text="Add / Edit Employee")
        form.pack(fill="x", pady=8)

        self.emp_id = tk.StringVar()
        self.name = tk.StringVar()
        self.dept = tk.StringVar()
        self.role = tk.StringVar()
        self.base = tk.StringVar()
        self.join_date = tk.StringVar(value=DEFAULT_DATE_STR)
        self.phone = tk.StringVar()
        self.email = tk.StringVar()

        # Register validators
        vcmd_digits = (self.register(self._vc_digits), '%P')
        vcmd_decimal = (self.register(self._vc_decimal), '%P')
        vcmd_phone = (self.register(self._vc_phone), '%P')

        # Row 1
        r1 = ttk.Frame(form)
        r1.pack(fill="x", pady=4)
        ttk.Label(r1, text="Employee ID* (numbers only)").grid(row=0, column=0, sticky="w", padx=(0,6))
        ttk.Entry(r1, textvariable=self.emp_id, width=18, validate='key', validatecommand=vcmd_digits).grid(row=0, column=1, sticky="w")
        ttk.Label(r1, text="Name*").grid(row=0, column=2, sticky="w", padx=(12,6))
        ttk.Entry(r1, textvariable=self.name).grid(row=0, column=3, sticky="ew")

        # Row 2
        r2 = ttk.Frame(form)
        r2.pack(fill="x", pady=4)
        ttk.Label(r2, text="Department").grid(row=0, column=0, sticky="w", padx=(0,6))
        ttk.Entry(r2, textvariable=self.dept).grid(row=0, column=1, sticky="ew")
        ttk.Label(r2, text="Role").grid(row=0, column=2, sticky="w", padx=(12,6))
        ttk.Entry(r2, textvariable=self.role).grid(row=0, column=3, sticky="ew")

        # Row 3
        r3 = ttk.Frame(form)
        r3.pack(fill="x", pady=4)
        ttk.Label(r3, text="Base Salary* (per month, numbers only)").grid(row=0, column=0, sticky="w", padx=(0,6))
        ttk.Entry(r3, textvariable=self.base, validate='key', validatecommand=vcmd_decimal).grid(row=0, column=1, sticky="ew")
        ttk.Label(r3, text="Join Date (YYYY-MM-DD)").grid(row=0, column=2, sticky="w", padx=(12,6))
        jd_frame = ttk.Frame(r3)
        jd_frame.grid(row=0, column=3, sticky="ew")
        ttk.Entry(jd_frame, textvariable=self.join_date).pack(side="left", fill="x", expand=True)
        ttk.Button(jd_frame, text="📅", command=lambda: CalendarPicker(self, self.join_date)).pack(side="left", padx=(3,0))


        # Row 4
        r4 = ttk.Frame(form)
        r4.pack(fill="x", pady=4)
        ttk.Label(r4, text="Phone (digits)").grid(row=0, column=0, sticky="w", padx=(0,6))
        ttk.Entry(r4, textvariable=self.phone, validate='key', validatecommand=vcmd_phone).grid(row=0, column=1, sticky="ew")
        ttk.Label(r4, text="Email").grid(row=0, column=2, sticky="w", padx=(12,6))
        ttk.Entry(r4, textvariable=self.email).grid(row=0, column=3, sticky="ew")

        for frm in (r1,r2,r3,r4):
            frm.columnconfigure(3, weight=1)

        # Buttons
        btns = ttk.Frame(form)
        btns.pack(fill="x", pady=(6,2))
        ttk.Button(btns, text="Save New", command=self.on_save_new).pack(side="left")
        ttk.Button(btns, text="Update", command=self.on_update).pack(side="left", padx=6)
        ttk.Button(btns, text="Clear", command=self.clear_form).pack(side="left")

        # Tree
        table = ttk.Frame(self)
        table.pack(fill="both", expand=True, pady=(8,0))
        self.tree = ttk.Treeview(table, columns=self.COLS, show="headings", height=10)
        for c in self.COLS:
            self.tree.heading(c, text=c)
            self.tree.column(c, width=110, anchor="w")
        self.tree.pack(side="left", fill="both", expand=True)
        self.tree.bind("<<TreeviewSelect>>", self.on_tree_select)

        vsb = ttk.Scrollbar(table, orient="vertical", command=self.tree.yview)
        self.tree.configure(yscroll=vsb.set)
        vsb.pack(side="right", fill="y")

    # Validation callbacks
    def _vc_digits(self, P):
        # allow empty during typing
        return P.isdigit() or P == ""

    def _vc_decimal(self, P):
        # allow only digits and at most one dot
        if P == "":
            return True
        if P.count(".") > 1:
            return False
        return all(ch.isdigit() or ch == '.' for ch in P)

    def _vc_phone(self, P):
        # digits only, allow empty while typing
        return P.isdigit() or P == ""

    def _refresh_tree(self, rows):
        self.tree.delete(*self.tree.get_children())
        for r in rows:
            self.tree.insert("", "end", values=(
                r.get("emp_id",""),
                r.get("name",""),
                r.get("department",""),
                r.get("role",""),
                r.get("base_salary",""),
                r.get("join_date",""),
                r.get("phone",""),
                r.get("email",""),
            ))

    def on_search(self):
        q = self.search_var.get().strip().lower()
        if not q:
            self._refresh_tree(self.records)
            return
        def match(r):
            return any(q in str(r.get(k,"")).lower() for k in ("emp_id","name","department","role"))
        self._refresh_tree(list(filter(match, self.records)))

    def on_export(self):
        if not self.records:
            messagebox.showinfo("Export", "No employees to export.")
            return
        path = filedialog.asksaveasfilename(
            defaultextension=".csv",
            initialfile="employees.csv",
            filetypes=[("CSV files","*.csv")]
        )
        if not path: return
        headers = ["Emp Id","Name","Department","Role","Base Salary","Join Date","Phone","Email"]
        rows = []
        for r in self.records:
            rows.append({
                "Emp Id": r.get("emp_id",""),
                "Name": r.get("name",""),
                "Department": r.get("department",""),
                "Role": r.get("role",""),
                "Base Salary": r.get("base_salary",""),
                "Join Date": r.get("join_date",""),
                "Phone": r.get("phone",""),
                "Email": r.get("email",""),
            })
        export_csv(path, headers, rows)
        messagebox.showinfo("Export", f"Exported to:\n{path}")

    def _read_form(self):
        rec = {
            "emp_id": self.emp_id.get().strip(),
            "name": self.name.get().strip(),
            "department": self.dept.get().strip(),
            "role": self.role.get().strip(),
            "base_salary": self.base.get().strip(),
            "join_date": self.join_date.get().strip(),
            "phone": self.phone.get().strip(),
            "email": self.email.get().strip(),
        }
        # Validation
        if not rec["emp_id"]:
            raise ValueError("Employee ID is required.")
        if not rec["emp_id"].isdigit():
            raise ValueError("Employee ID must be numeric.")
        if not rec["name"]:
            raise ValueError("Name is required.")
        try:
            bs = float(rec["base_salary"])
            if bs < 0: raise ValueError
        except:
            raise ValueError("Base Salary must be a positive number.")
        if rec["join_date"] and not valid_date(rec["join_date"]):
            raise ValueError("Join Date must be YYYY-MM-DD.")
        if rec["email"] and not EMAIL_RE.match(rec["email"]):
            raise ValueError("Email format is invalid.")
        if rec["phone"] and not rec["phone"].isdigit():
            raise ValueError("Phone must contain digits only.")
        return rec

    def clear_form(self):
        self.emp_id.set("")
        self.name.set("")
        self.dept.set("")
        self.role.set("")
        self.base.set("")
        self.join_date.set(DEFAULT_DATE_STR)
        self.phone.set("")
        self.email.set("")
        self.tree.selection_remove(self.tree.selection())

    def on_save_new(self):
        try:
            rec = self._read_form()
        except ValueError as e:
            messagebox.showerror("Validation", str(e))
            return
        if any(r["emp_id"] == rec["emp_id"] for r in self.records):
            messagebox.showerror("Duplicate", "Employee ID already exists.")
            return
        # normalize base_salary to float
        rec["base_salary"] = float(rec["base_salary"])
        self.records.append(rec)
        save_employees(self.records)
        self._refresh_tree(self.records)
        messagebox.showinfo("Saved", "Employee saved.")
        self.clear_form()

    def on_update(self):
        try:
            rec = self._read_form()
        except ValueError as e:
            messagebox.showerror("Validation", str(e))
            return
        idx = None
        for i, r in enumerate(self.records):
            if r["emp_id"] == rec["emp_id"]:
                idx = i
                break
        if idx is None:
            messagebox.showerror("Not Found", "Employee ID not found. Use Save New to add.")
            return
        rec["base_salary"] = float(rec["base_salary"])
        self.records[idx] = rec
        save_employees(self.records)
        self._refresh_tree(self.records)
        messagebox.showinfo("Updated", "Employee updated.")
        self.clear_form()

    def on_delete_selected(self):
        sel = self.tree.selection()
        if not sel:
            messagebox.showinfo("Delete", "Select at least one row in the table.")
            return
        ids = []
        for it in sel:
            vals = self.tree.item(it, "values")
            if vals: ids.append(vals[0])
        if not ids: return
        if not messagebox.askyesno("Confirm", f"Delete {len(ids)} employee(s)?"):
            return
        self.records = [r for r in self.records if r.get("emp_id") not in ids]
        save_employees(self.records)
        self._refresh_tree(self.records)
        self.clear_form()

    def on_tree_select(self, _evt):
        sel = self.tree.selection()
        if not sel: return
        vals = self.tree.item(sel[0], "values")
        # Map to form
        self.emp_id.set(vals[0])
        self.name.set(vals[1])
        self.dept.set(vals[2])
        self.role.set(vals[3])
        self.base.set(str(vals[4]))
        self.join_date.set(vals[5])
        self.phone.set(vals[6])
        self.email.set(vals[7])


# ---------------------- Attendance Tab ----------------------

class AttendanceTab(ttk.Frame):
    COLS = ("Emp_Id", "Name", "Status (P/A)", "IP", "Overtime Hours")

    def __init__(self, master):
        super().__init__(master, padding=10)
        self.date_str = tk.StringVar(value=DEFAULT_DATE_STR)
        self.rows = []  # current loaded rows
        self._build_ui()

    def _build_ui(self):
        header = ttk.Frame(self)
        header.pack(fill="x", pady=(0,8))
        ttk.Label(header, text="Attendance", font=("Segoe UI", 11, "bold")).pack(side="left")

        bar = ttk.Frame(self)
        bar.pack(fill="x", pady=(0,6))
        ttk.Label(bar, text="Date (YYYY-MM-DD):").pack(side="left")
        ttk.Entry(bar, textvariable=self.date_str, width=14).pack(side="left", padx=6)
        ttk.Button(bar, text="📅", command=lambda: CalendarPicker(self, self.date_str)).pack(side="left")
        ttk.Button(bar, text="Load", command=self.on_load).pack(side="left")
        ttk.Button(bar, text="Mark All Present", command=lambda: self.mark_all("P")).pack(side="left", padx=6)
        ttk.Button(bar, text="Mark All Absent", command=lambda: self.mark_all("A")).pack(side="left")
        ttk.Button(bar, text="Export", command=self.on_export).pack(side="left", padx=6)

        table = ttk.Frame(self)
        table.pack(fill="both", expand=True, pady=(8,6))
        self.tree = ttk.Treeview(table, columns=self.COLS, show="headings", height=11)
        for c in self.COLS:
            self.tree.heading(c, text=c)
            w = 120 if c != "Name" else 180
            self.tree.column(c, width=w, anchor="w")
        self.tree.pack(side="left", fill="both", expand=True)
        vsb = ttk.Scrollbar(table, orient="vertical", command=self.tree.yview)
        self.tree.configure(yscroll=vsb.set)
        vsb.pack(side="right", fill="y")

        # Edit panel for selected row
        editor = ttk.LabelFrame(self, text="Edit Selected")
        editor.pack(fill="x", pady=(6,2))
        self.sel_emp_id = tk.StringVar()
        self.sel_name = tk.StringVar()
        self.sel_status = tk.StringVar()
        self.sel_ip = tk.StringVar(value=get_host_ip())
        self.sel_ot = tk.StringVar(value="0")

        r1 = ttk.Frame(editor); r1.pack(fill="x", pady=3)
        ttk.Label(r1, text="Emp ID").grid(row=0, column=0, sticky="w")
        ttk.Entry(r1, textvariable=self.sel_emp_id, state="readonly", width=16).grid(row=0, column=1, padx=(6,12))
        ttk.Label(r1, text="Name").grid(row=0, column=2, sticky="w")
        ttk.Entry(r1, textvariable=self.sel_name, state="readonly").grid(row=0, column=3, sticky="ew", padx=(6,12))
        r1.columnconfigure(3, weight=1)

        r2 = ttk.Frame(editor); r2.pack(fill="x", pady=3)
        ttk.Label(r2, text="Status (P/A)").grid(row=0, column=0, sticky="w")
        ttk.Combobox(r2, textvariable=self.sel_status, values=("P","A"), width=8).grid(row=0, column=1, padx=(6,12))
        ttk.Label(r2, text="IP").grid(row=0, column=2, sticky="w")
        ttk.Entry(r2, textvariable=self.sel_ip, width=16).grid(row=0, column=3, padx=(6,12))
        ttk.Label(r2, text="Overtime Hours").grid(row=0, column=4, sticky="w")
        # validate OT numeric
        vcmd_decimal = (self.register, '%P')
        ttk.Entry(r2, textvariable=self.sel_ot, width=10, validate='key', validatecommand=vcmd_decimal).grid(row=0, column=5, padx=(6,12))
        ttk.Button(editor, text="Save", command=self.on_save_selected).pack(side="left", padx=6, pady=(4,6))
        ttk.Button(editor, text="Save All to File", command=self.on_save_all).pack(side="left", padx=6, pady=(4,6))

        self.tree.bind("<<TreeviewSelect>>", self.on_tree_select)

    def on_load(self):
        day = self.date_str.get().strip()
        if not valid_date(day):
            messagebox.showerror("Date", "Please enter a valid date in YYYY-MM-DD.")
            return
        # Build rows based on employees (if first time), else load existing
        existing = {r["emp_id"]: r for r in load_attendance(day)}
        emps = load_employees()
        rows = []
        for e in emps:
            emp_id = e.get("emp_id","")
            rows.append({
                "Emp_Id": emp_id,
                "Name": e.get("name",""),
                "Status (P/A)": existing.get(emp_id, {}).get("status", "A"),
                "IP": existing.get(emp_id, {}).get("ip", get_host_ip()),
                "Overtime Hours": existing.get(emp_id, {}).get("overtime", 0),
            })
        self.rows = rows
        self._refresh_tree()
        messagebox.showinfo("Loaded", f"Attendance loaded for {day}.")

    def _refresh_tree(self):
        self.tree.delete(*self.tree.get_children())
        for r in self.rows:
            self.tree.insert("", "end", values=(r["Emp_Id"], r["Name"], r["Status (P/A)"], r["IP"], r["Overtime Hours"]))

    def mark_all(self, status):
        if not self.rows: return
        for r in self.rows:
            r["Status (P/A)"] = status
        self._refresh_tree()

    def on_tree_select(self, _evt):
        sel = self.tree.selection()
        if not sel: return
        vals = self.tree.item(sel[0], "values")
        self.sel_emp_id.set(vals[0])
        self.sel_name.set(vals[1])
        self.sel_status.set(vals[2])
        self.sel_ip.set(vals[3])
        self.sel_ot.set(str(vals[4]))

    def on_save_selected(self):
        emp_id = self.sel_emp_id.get()
        if not emp_id: return
        # validate OT numeric
        try:
            ot = float(self.sel_ot.get())
            if ot < 0: raise ValueError
        except:
            messagebox.showerror("Overtime", "Overtime Hours must be a non-negative number.")
            return
        for r in self.rows:
            if r["Emp_Id"] == emp_id:
                r["Status (P/A)"] = (self.sel_status.get() or "A").upper()[0]
                r["IP"] = self.sel_ip.get().strip() or get_host_ip()
                r["Overtime Hours"] = ot
                break
        self._refresh_tree()
        messagebox.showinfo("Saved", "Selected row updated (not yet written to file).")

    def on_save_all(self):
        day = self.date_str.get().strip()
        if not valid_date(day):
            messagebox.showerror("Date", "Please enter a valid date in YYYY-MM-DD.")
            return
        # Convert to persistable structure
        persist = []
        for r in self.rows:
            persist.append({
                "emp_id": r["Emp_Id"],
                "name": r["Name"],
                "status": r["Status (P/A)"],
                "ip": r["IP"],
                "overtime": float(r["Overtime Hours"]) if str(r["Overtime Hours"]).strip() != "" else 0.0
            })
        save_attendance(day, persist)
        messagebox.showinfo("Saved", f"Attendance saved for {day}.")

    def on_export(self):
        day = self.date_str.get().strip()
        if not valid_date(day):
            messagebox.showerror("Date", "Please enter a valid date in YYYY-MM-DD.")
            return
        rows = load_attendance(day)
        if not rows:
            messagebox.showinfo("Export", f"No attendance saved for {day}.")
            return
        path = filedialog.asksaveasfilename(
            defaultextension=".csv",
            initialfile=f"attendance-{day}.csv",
            filetypes=[("CSV files","*.csv")]
        )
        if not path: return
        headers = ["emp_id","name","status","ip","overtime"]
        export_csv(path, headers, rows)
        messagebox.showinfo("Export", f"Exported to:\n{path}")


# ---------------------- Salary Tab ----------------------

class SalaryTab(ttk.Frame):
    COLS = ("Emp_Id", "Name", "Month", "Base Salary", "Working Days", "Present", "Absent", "Overtime Hours", "Net Pay")

    def __init__(self, master):
        super().__init__(master, padding=10)
        self.month_var = tk.StringVar(value="2025-08")
        self.emp_var = tk.StringVar()  # empty => ALL
        self.rows = []
        self._build_ui()

    def _build_ui(self):
        header = ttk.Frame(self)
        header.pack(fill="x", pady=(0,8))
        ttk.Label(header, text="Salary", font=("Segoe UI", 11, "bold")).pack(side="left")

        bar = ttk.Frame(self); bar.pack(fill="x", pady=(0,6))
        ttk.Label(bar, text="Employee ID (leave blank for ALL)").pack(side="left")
        ttk.Entry(bar, textvariable=self.emp_var, width=18).pack(side="left", padx=6)
        ttk.Label(bar, text="Month").pack(side="left")
        # Provide month name combobox and year spinbox for easier selection
        self.month_name = tk.StringVar(value=calendar.month_name[int(self.month_var.get().split('-')[1])] if '-' in self.month_var.get() else 'August')
        self.year_spin = tk.IntVar(value=int(self.month_var.get().split('-')[0]))
        months = [calendar.month_name[i] for i in range(1,13)]
        ttk.Combobox(bar, values=months, textvariable=self.month_name, width=10).pack(side="left", padx=(6,2))
        ttk.Spinbox(bar, from_=1900, to=3000, textvariable=self.year_spin, width=6).pack(side="left", padx=(2,6))
        ttk.Button(bar, text="Calculate", command=self.on_calculate).pack(side="left")
        ttk.Button(bar, text="Export", command=self.on_export).pack(side="left", padx=6)

        table = ttk.Frame(self)
        table.pack(fill="both", expand=True, pady=(8,0))
        self.tree = ttk.Treeview(table, columns=self.COLS, show="headings", height=12)
        for c in self.COLS:
            self.tree.heading(c, text=c)
            w = 110
            if c in ("Name",): w = 180
            if c in ("Net Pay","Base Salary"): w = 120
            self.tree.column(c, width=w, anchor="w")
        self.tree.pack(side="left", fill="both", expand=True)
        vsb = ttk.Scrollbar(table, orient="vertical", command=self.tree.yview)
        self.tree.configure(yscroll=vsb.set)
        vsb.pack(side="right", fill="y")

    def _refresh(self):
        self.tree.delete(*self.tree.get_children())
        for r in self.rows:
            self.tree.insert("", "end", values=(
                r["Emp_Id"], r["Name"], r["Month"], f'{r["Base Salary"]:.2f}',
                r["Working Days"], r["Present"], r["Absent"],
                f'{r["Overtime Hours"]:.2f}', f'{r["Net Pay"]:.2f}'
            ))

    def _gather_month_attendance(self, year, month):
        """Return dict emp_id -> dict(present_days, absent_days, ot_hours_total)."""
        d = {}
        for day in month_days_iter(year, month):
            rows = load_attendance(day)
            if not rows:  # if no file for the day, skip
                continue
            for r in rows:
                emp = r.get("emp_id","")
                if emp not in d:
                    d[emp] = {"present":0, "absent":0, "ot":0.0}
                status = (r.get("status","A") or "A").upper()
                if status.startswith("P"):
                    d[emp]["present"] += 1
                else:
                    d[emp]["absent"] += 1
                try:
                    d[emp]["ot"] += float(r.get("overtime", 0.0))
                except:
                    pass
        return d

    def on_calculate(self):
        # build month string from widgets
        month_name = self.month_name.get()
        try:
            month_idx = list(calendar.month_name).index(month_name)
        except ValueError:
            messagebox.showerror("Month", "Select a valid month.")
            return
        year = int(self.year_spin.get())
        month = month_idx
        month_str = f"{year:04d}-{month:02d}"
        self.month_var.set(month_str)

        # Accurate working days: count business days (Mon-Fri)
        working_days = business_days_in_month(year, month)

        att = self._gather_month_attendance(year, month)
        emps = load_employees()
        emp_filter = self.emp_var.get().strip()
        rows = []

        for e in emps:
            emp_id = e.get("emp_id","")
            if emp_filter and emp_id != emp_filter:
                continue
            name = e.get("name","")
            base = float(e.get("base_salary", 0.0))
            stats = att.get(emp_id, {"present":0,"absent":0,"ot":0.0})
            present = stats["present"]
            absent = stats["absent"]
            ot_hours = stats["ot"]

            # Salary model:
            # - Prorate by business days present vs working_days
            # - OT: base/176 per hour (standard) -> can be adjusted
            per_hour = base / 176.0 if base else 0.0
            prorated = base * (present / float(working_days) if working_days > 0 else 0.0)
            ot_pay = per_hour * ot_hours
            net = max(0.0, prorated + ot_pay)

            rows.append({
                "Emp_Id": emp_id,
                "Name": name,
                "Month": month_str,
                "Base Salary": base,
                "Working Days": working_days,
                "Present": present,
                "Absent": absent,
                "Overtime Hours": ot_hours,
                "Net Pay": net
            })

        self.rows = rows
        if not rows:
            messagebox.showinfo("Salary", "No employees or no matching Employee ID.")
        self._refresh()

    def on_export(self):
        if not self.rows:
            messagebox.showinfo("Export", "No salary results to export. Calculate first.")
            return
        month = self.month_var.get().strip()
        default = os.path.join(SAL_DIR, f"salary-{month}.csv") if valid_month(month) else "salary.csv"
        path = filedialog.asksaveasfilename(
            defaultextension=".csv",
            initialfile=os.path.basename(default),
            initialdir=(SAL_DIR if os.path.isdir(SAL_DIR) else os.getcwd()),
            filetypes=[("CSV files","*.csv")]
        )
        if not path: return
        headers = list(self.COLS)
        export_csv(path, headers, self.rows)
        messagebox.showinfo("Export", f"Exported to:\n{path}")

# ---------------------- Main App ----------------------

class App(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("Employee Management System")
        self.geometry("1024x640")
        self._style()
        self.notebook = None
        self._show_login()

    def _style(self):
        style = ttk.Style(self)
        # Use a clean built-in theme
        try:
            style.theme_use("clam")
        except:
            pass
        style.configure("Treeview", rowheight=24)
        style.configure("TButton", padding=6)
        style.configure("TEntry", padding=4)

    def _show_login(self):
        self.withdraw()
        login = LoginWindow(self, on_success=self._on_login_success)
        self.wait_window(login)

    def _on_login_success(self):
        self.deiconify()
        self._build_tabs()

    def _build_tabs(self):
        self.notebook = ttk.Notebook(self)
        self.notebook.pack(fill="both", expand=True)

        self.tab_employees = EmployeesTab(self.notebook)
        self.tab_attendance = AttendanceTab(self.notebook)
        self.tab_salary = SalaryTab(self.notebook)

        self.notebook.add(self.tab_employees, text="Employees")
        self.notebook.add(self.tab_attendance, text="Attendance")
        self.notebook.add(self.tab_salary, text="Salary")

if __name__ == "__main__":
    ensure_dirs()
    app = App()
    app.mainloop()
