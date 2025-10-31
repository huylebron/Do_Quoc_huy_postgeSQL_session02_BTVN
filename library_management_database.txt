-- quản lý thư viên 

CREATE TABLE members (
                         member_id      BIGSERIAL PRIMARY KEY,
                         full_name      VARCHAR(150)          NOT NULL,
                         email          VARCHAR(255)          NOT NULL,
                         phone          VARCHAR(25),
                         date_of_birth  DATE,
                         address        TEXT,
                         status         VARCHAR(10)           NOT NULL CHECK (status IN ('active','inactive')),
                         joined_at      DATE                  NOT NULL DEFAULT CURRENT_DATE,
    -- check trùng email
                         CONSTRAINT uq_members_email UNIQUE (email)
);

insert into members ( full_name, email, phone, date_of_birth, address, status, joined_at)
values
    ('Nguyễn Văn A', 'a@example.com', '0909123456', '1995-05-12', 'Hà Nội', 'active', '2023-01-15'),
    ('Trần Thị B', 'b@example.com', '0912345678', '1998-09-22', 'Hồ Chí Minh', 'active', '2023-02-10'),
    ('Lê Văn C', 'c@example.com', '0988888888', '2000-12-02', 'Đà Nẵng', 'inactive', '2022-11-01'),
    ('Phạm Minh D', 'd@example.com', '0934567890', '1992-03-17', 'Hải Phòng', 'active', '2023-03-12'),
    ('Đỗ Thị E', 'e@example.com', '0977777777', '2001-07-09', 'Cần Thơ', 'active', '2024-05-05');

select * from members;

create table authors
(
    author_id bigserial primary key,
    name      varchar(150) not null unique,
    bio       text
);
--  tac gia
insert into authors ( name, bio)
values
    ('Isaac Newton', 'Nhà vật lý và toán học người Anh, cha đẻ của cơ học cổ điển.'),
    ('Stephen Hawking', 'Nhà vật lý lý thuyết, tác giả của "Lược sử thời gian".'),
    ('Haruki Murakami', 'Nhà văn Nhật Bản nổi tiếng với phong cách siêu thực.'),
    ('J.K. Rowling', 'Tác giả bộ truyện Harry Potter.'),
    ('Nguyễn Nhật Ánh', 'Nhà văn Việt Nam chuyên viết cho thiếu nhi và tuổi mới lớn.');

select * from authors;

-- categories

create table categories
(
    category_id serial primary key,
    name        varchar(150) not null unique,
    description text
);

insert into categories (name, description)
values ('Science Fiction', 'Loại tiểu thuyết về khoa học và tương lai.'),
       ('Fantasy', 'Loại tiểu thuyết về những câu chuyện thần thoại và kỳ ảo.'),
       ('Mystery', 'Loại tiểu thuyết về các vụ án bí ẩn và bí mật.'),
       ('Romance', 'Loại tiểu thuyết về tình yêu và mối quan hệ.'),
       ('Thriller', 'Loại tiểu thuyết về các vụ án và sự giật mình.');

select *
from categories;

-- nha xuat ban

create table publishers
(
    publisher_id serial primary key,
    name         varchar(150) not null unique,
    address      text
);

insert into publishers (name, address)
values ('Nhà Xuất Bản A', 'Hà Nội'),
       ('Nhà Xuất Bản B', 'Hồ Chí Minh'),
       ('Nhà Xuất Bản C', 'Đà Nẵng'),
       ('Nhà Xuất Bản D', 'Hải Phòng'),
       ('Nhà Xuất Bản E', 'Cần Thơ');

select *
from publishers;

-- sach

create table books (
    book_id bigserial primary key,
    code varchar(30) not null unique, -- ma sach
    title varchar(255) not null,
    publish_year integer not null,
    publisher_id integer references publishers(publisher_id) on delete restrict,
    category_id integer not null references categories(category_id) on delete restrict
);

insert into books (code, title, publish_year, publisher_id, category_id)
values
    ('ISBN-1234567890', 'Toán học cổ điển của Newton', 1687, 1, 1),
    ('ISBN-9876543210', 'A Brief History of Time', 1988, 2, 2),
    ('ISBN-1111111111', 'Kafka on the Shore', 2002, 3, 3),
    ('ISBN-2222222222', 'Harry Potter and the Philosopher''s Stone', 1997, 4, 4),
    ('ISBN-3333333333', 'One Hundred Years of Solitude', 1967, 5, 5);

select * from books;

-- sach va tac gia : n - n

create table book_authors (
    book_id bigint not null references books(book_id) ,
    author_id bigint not null references authors(author_id),
    primary key (book_id, author_id)
);
insert into book_authors (book_id, author_id)
values
    (1, 1),
    (2, 2),
    (3, 3),
    (4, 4),
    (5, 5);

select *
from book_authors;

-- ban sao cua sach ( book copies )
create table book_copies (

    copy_id bigserial primary key,
    book_id bigint not null references books(book_id),
    status varchar(50) not null check (status in ('available', 'borrowed', 'lost')),
    created_at timestamp default current_timestamp
);
insert into book_copies (book_id, status)
values
    (1, 'available'),
    (1, 'available'),
    (2, 'available'),
    (3, 'available'),
    (4, 'available'),
    (5, 'available');

select *
from book_copies;

-- phieu muon ( loan )
create table loans
(
    loan_id   bigserial primary key,
    member_id bigint not null references members (member_id),
    load_date date default current_date,
    due_date  date   not null,
    constraint _load_date_check check (due_date >= load_date)



);

insert into loans (member_id, due_date , load_date)
values
    (1, '2024-05-15', '2024-05-01'),
    (2, '2024-06-14', '2024-06-01'),
    (3, '2024-07-15', '2024-07-01'),
    (4, '2024-08-24', '2024-08-10'),
    (5, '2024-10-04', '2024-09-20');

select *
from loans;
 -- chi tiet phieu muon ( loan items )

 create table loan_items (
     loan_item_id bigserial primary key,
     loan_id bigint not null references loans(loan_id),
     copy_id bigint not null references book_copies(copy_id),
     borrowed_at timestamp default current_timestamp,
     due_at timestamp not null,
     constraint _due_at_check check (due_at >= borrowed_at)

 );

INSERT INTO loan_items (loan_id, copy_id, borrowed_at, due_at)
VALUES
    (8, 1, DATE '2024-05-01', DATE '2024-05-15'),
    (9, 2, DATE '2024-06-01', DATE '2024-06-14'),
    (10, 3, DATE '2024-07-01', DATE '2024-07-15'),
    (11, 4, DATE '2024-08-10', DATE '2024-08-24'),
    (12, 5, DATE '2024-09-20', DATE '2024-10-04');


select * from loan_items;
























