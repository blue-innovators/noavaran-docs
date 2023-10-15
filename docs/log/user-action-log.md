# ثبت userAction ها

<p>
 برای ثبت یوزر اکشن های یک سیستم باید از کامند AppendUserActionLogCommand در نرم افزار کامان استفاده کرد . 
 این کامند شامل فیلدهای زیر است:
</p>

```cs
  {
        public long ReferenceId { get; set; }
        public string ReferenceType { get; set; }
        public string RequestType { get; set; }
        public long UserId { get; set; }
        public string UserName { get; set; }
        public UserType UserType { get; set; }
        public string UserRole { get; set; }
        public string UserTokenId { get; set; }
        public long? BusinessId { get; set; }
        public long? BranchId { get; set; }
    }
``` 