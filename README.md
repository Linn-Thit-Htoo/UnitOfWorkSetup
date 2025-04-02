## IUnitOfWork
```csharp
    public interface IUnitOfWork
    {
        void SaveChanges();
        Task SaveChangesAsync(CancellationToken cs = default);
    }
```

## UnitOfWork
```csharp
    public class UnitOfWork : IUnitOfWork
    {
        internal readonly DbContext _context;
        internal readonly string? _currentUser = "SYSTEM";
        internal readonly IHttpContextAccessor _httpContextAccessor;

        public UnitOfWork(AppDbContext context, IHttpContextAccessor httpContextAccessor)
        {
            _context = context;
            _httpContextAccessor = httpContextAccessor;

            if (_httpContextAccessor.HttpContext is not null)
            {
                if (_httpContextAccessor.HttpContext.User.Identity!.IsAuthenticated)
                {
                    _currentUser = _httpContextAccessor
                        .HttpContext.User.Identities.FirstOrDefault()!
                        .Claims.Where(x => x.Type == ClaimTypes.NameIdentifier)
                        .FirstOrDefault()!
                        .Value;
                }
            }
        }

        public void SaveChanges()
        {
            var modifiedEntries = _context
                .ChangeTracker.Entries()
                .Where(x => x.State == EntityState.Modified || x.State == EntityState.Added)
                .ToList();

            foreach (var entry in modifiedEntries)
            {
                Type type = entry.Entity.GetType();

                if (entry.State == EntityState.Added)
                {
                    PropertyInfo createdBy = type.GetProperty("CreatedBy")!;
                    createdBy?.SetValue(entry.Entity, _currentUser);

                    PropertyInfo createdDate = type.GetProperty("CreatedAt")!;
                    createdDate?.SetValue(entry.Entity, DateTime.Now);
                }

                if (entry.State == EntityState.Modified)
                {
                    PropertyInfo modifiedBy = type.GetProperty("ModifiedBy")!;
                    modifiedBy?.SetValue(entry.Entity, _currentUser);

                    PropertyInfo modifiedAt = type.GetProperty("ModifiedAt")!;
                    modifiedAt?.SetValue(entry.Entity, DateTime.Now);
                }
            }

            _context.SaveChanges();
        }

        public async Task SaveChangesAsync(CancellationToken cs = default)
        {
            var modifiedEntries = _context
                .ChangeTracker.Entries()
                .Where(x => x.State == EntityState.Modified || x.State == EntityState.Added)
                .ToList();

            foreach (var entry in modifiedEntries)
            {
                Type type = entry.Entity.GetType();

                if (entry.State == EntityState.Added)
                {
                    PropertyInfo createdBy = type.GetProperty("CreatedBy")!;
                    createdBy?.SetValue(entry.Entity, _currentUser);

                    PropertyInfo createdDate = type.GetProperty("CreatedAt")!;
                    createdDate?.SetValue(entry.Entity, DateTime.Now);
                }

                if (entry.State == EntityState.Modified)
                {
                    PropertyInfo modifiedBy = type.GetProperty("ModifiedBy")!;
                    modifiedBy?.SetValue(entry.Entity, _currentUser);

                    PropertyInfo modifiedAt = type.GetProperty("ModifiedAt")!;
                    modifiedAt?.SetValue(entry.Entity, DateTime.Now);
                }
            }

            await _context.SaveChangesAsync(cs);
        }
    }
```
