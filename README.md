# oracleapextree
Dynamic Hierarchy Tree in Oracle APEX


This repository provides a complete guide to creating a Dynamic Hierarchy Tree in Oracle APEX. The tree supports features like:

Expand/Collapse categories and items.
A search bar for filtering categories and items.
Interactive and responsive design using PL/SQL, JavaScript, and CSS.

**Table of Contents**
Database Tables
PL/SQL Function for Tree Generation
HTML for Search and Buttons
JavaScript for Functionality
CSS for Styling
How to Use
Preview

Table
------------------------------

CREATE TABLE "IN_CATEGORY" 
(
    "CT_ID" NUMBER, 
    "CT_NAME" VARCHAR2(50), 
    "PARENT_CODE" NUMBER, 
    "CT_CODE" VARCHAR2(100), 
    PRIMARY KEY ("CT_ID")
);

 CREATE TABLE "IN_ITEM" 
   (	"ITEM_NO" NUMBER(30,0), 
	"ITEM_NAME" VARCHAR2(100) NOT NULL ENABLE, 
	"ITEM_ID" VARCHAR2(50), 
	"CT_ID" NUMBER
   ) ;

  ALTER TABLE "IN_ITEM" ADD CONSTRAINT "FK_ITEM_CATEGORY" FOREIGN KEY ("CT_ID")
	  REFERENCES "IN_CATEGORY" ("CT_ID") ENABLE;

**   PL/SQL Function for Tree Generation**
This PL/SQL function recursively builds the HTML structure for the hierarchy tree.

DECLARE
    v_html CLOB;

    PROCEDURE build_tree(p_parent_id IN NUMBER) IS
    BEGIN
        FOR cat_rec IN (
            SELECT c.CT_ID, c.CT_NAME, c.CT_CODE
            FROM IN_CATEGORY c
            WHERE NVL(c.PARENT_CODE, 0) = p_parent_id
            ORDER BY c.CT_ID
        )
        LOOP
            v_html := v_html || '<li class="category" data-category-id="' || cat_rec.CT_ID || '">';
            v_html := v_html || '<span class="toggle">+</span> ' || cat_rec.CT_CODE || '-' || cat_rec.CT_NAME;
            v_html := v_html || '<ul class="nested">';

            FOR item_rec IN (
                SELECT i.ITEM_NAME, i.ITEM_ID, i.ITEM_NO 
                FROM IN_ITEM i
                WHERE i.CT_ID = cat_rec.CT_ID
                ORDER BY i.ITEM_NO, i.ITEM_ID ASC
            )
            LOOP
                v_html := v_html || '<li class="item">' || item_rec.ITEM_ID || ' - ' || item_rec.ITEM_NAME || '</li>';
            END LOOP;

            build_tree(cat_rec.CT_ID);
            v_html := v_html || '</ul></li>';
        END LOOP;
    END;

BEGIN
    v_html := '<ul id="categoryTree">';
    build_tree(0);
    v_html := v_html || '</ul>';
    RETURN v_html;
END;


**HTML for Search and Buttons**
Add the following HTML in the Header or Footer region of Oracle APEX for search and expand/collapse buttons:

<div style="display: flex; align-items: center; justify-content: space-between; margin-bottom: 15px;">
    <!-- Search Bar -->
    <div style="position: relative; flex: 1; margin-right: 10px;">
        <input 
            type="text" 
            id="treeSearch" 
            placeholder="🔍 Search categories or items...(ابحث عن الفئات أو العناصر...)" 
            style="width: 100%; padding: 12px 40px 12px 16px; border: 1px solid #ccc; border-radius: 25px; font-size: 14px; background: #f9f9f9; box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.1); outline: none; transition: all 0.3s ease;"
        />
        <span id="clearSearch" style="position: absolute; top: 50%; right: 16px; transform: translateY(-50%); cursor: pointer; color: #888; font-size: 16px; display: none;">✖</span>
    </div>

    <!-- Buttons -->
    <div style="display: flex; gap: 10px;">
        <button id="expandAll" type="button" style="padding: 10px 16px; background: linear-gradient(135deg, #007bff, #0056b3); color: white; font-size: 14px; border: none; border-radius: 6px; cursor: pointer; box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1); transition: all 0.3s ease;">Expand</button>
        <button id="collapseAll" type="button" style="padding: 10px 16px; background: linear-gradient(135deg, #6c757d, #495057); color: white; font-size: 14px; border: none; border-radius: 6px; cursor: pointer; box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1); transition: all 0.3s ease;">Collapse</button>
    </div>
</div>


**JavaScript for Functionality**
Add this JavaScript to handle expand/collapse and search functionality:

document.addEventListener('DOMContentLoaded', function () {
    document.querySelectorAll('#categoryTree .toggle').forEach(function (toggle) {
        toggle.addEventListener('click', function () {
            const nested = this.parentElement.querySelector('.nested');
            if (nested) {
                nested.classList.toggle('active');
                this.textContent = nested.classList.contains('active') ? '−' : '+';
            }
        });
    });

    document.getElementById('expandAll').addEventListener('click', function (event) {
        event.preventDefault();
        document.querySelectorAll('#categoryTree .nested').forEach(nested => nested.classList.add('active'));
        document.querySelectorAll('#categoryTree .toggle').forEach(toggle => toggle.textContent = '−');
    });

    document.getElementById('collapseAll').addEventListener('click', function (event) {
        event.preventDefault();
        document.querySelectorAll('#categoryTree .nested').forEach(nested => nested.classList.remove('active'));
        document.querySelectorAll('#categoryTree .toggle').forEach(toggle => toggle.textContent = '+');
    });

    const searchInput = document.getElementById('treeSearch');
    searchInput.addEventListener('input', function () {
        const query = searchInput.value.toLowerCase();
        document.querySelectorAll('#categoryTree li').forEach(function (item) {
            const text = item.textContent.toLowerCase();
            item.style.display = text.includes(query) ? 'flex' : 'none';
        });
    });
});


**CSS for Styling**
Use this CSS to make the tree visually appealing and interactive:

/* Professional Button Styles */
#expandAll, #collapseAll {
    font-family: 'Roboto', Arial, sans-serif;
    padding: 12px 20px;
    font-size: 14px;
    font-weight: bold;
    border-radius: 6px;
    border: none;
    cursor: pointer;
    transition: all 0.3s ease;
    color: white;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

/* Expand All Button */
#expandAll {
    background: linear-gradient(135deg, #007bff, #0056b3); /* Professional blue gradient */
}

/* Collapse All Button */
#collapseAll {
    background: linear-gradient(135deg, #6c757d, #495057); /* Neutral gray gradient */
}

/* Hover Effects */
#expandAll:hover {
    background: linear-gradient(135deg, #0056b3, #003f7f); /* Darker blue on hover */
    box-shadow: 0 6px 12px rgba(0, 0, 0, 0.2);
    transform: translateY(-2px);
}

#collapseAll:hover {
    background: linear-gradient(135deg, #495057, #343a40); /* Darker gray on hover */
    box-shadow: 0 6px 12px rgba(0, 0, 0, 0.2);
    transform: translateY(-2px);
}

/* Active Effects */
#expandAll:active, #collapseAll:active {
    transform: translateY(2px); /* Button pressed down effect */
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}


/* Enhanced Search Bar */
#treeSearch {
    font-family: 'Roboto', Arial, sans-serif;
    font-size: 14px;
    width: 100%;
    padding: 12px 40px 12px 16px; /* Space for icon and text */
    border: 1px solid #ddd;
    border-radius: 25px;
    background: #f9f9f9;
    box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.1);
    transition: all 0.3s ease;
}

#treeSearch:focus {
    border-color: #007bff;
    background: #ffffff;
    box-shadow: 0 0 8px rgba(0, 123, 255, 0.2);
    outline: none;
}

/* Clear Button for Search */
#clearSearch {
    font-size: 16px;
    color: #888;
    cursor: pointer;
    transition: color 0.3s ease;
}

#clearSearch:hover {
    color: #ff5252;
}



/* General tree styling */
#categoryTree {
    font-family: 'Roboto', Arial, sans-serif; /* Use a clean, professional font */
    font-size: 14px;
    list-style: none;
    margin: 0;
    padding: 10px;
    color: #333;
    background: #fff;
    border: 1px solid #ddd;
    border-radius: 6px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

/* Styling the category and item containers */
#categoryTree li {
    margin: 5px 0;
    padding: 5px;
    display: flex;
    align-items: center;
    border-radius: 4px;
    transition: background-color 0.2s ease-in-out;
}

/* Hover effect for better interactivity */
#categoryTree li:hover {
    background-color: #f5f5f5;
}

/* Toggle button for expand/collapse */
#categoryTree .toggle {
    cursor: pointer;
    font-weight: bold;
    color: #007bff;
    margin-right: 8px;
    display: flex;
    align-items: center;
    justify-content: center;
    width: 20px;
    height: 20px;
    border: 1px solid #007bff;
    border-radius: 50%;
    font-size: 12px;
    line-height: 18px;
}

/* Icons for categories and items */
#categoryTree .category::before,
#categoryTree .item::before {
    content: '';
    display: inline-block;
    width: 20px;
    height: 20px;
    margin-right: 10px;
    background-size: contain;
}

/* Folder icon for categories */
#categoryTree .category::before {
    background-image: url('https://cdn-icons-png.flaticon.com/512/25/25694.png'); /* Folder icon */
    background-size: 16px;
    background-repeat: no-repeat;
}

/* File icon for items */
#categoryTree .item::before {
    background-image: url('https://cdn-icons-png.flaticon.com/512/1828/1828884.png'); /* File icon */
    background-size: 16px;
    background-repeat: no-repeat;
}

/* Nested lists (subcategories and items) */
#categoryTree .nested {
    display: none; /* Initially hidden */
    list-style: none;
    padding-left: 20px;
    border-left: 1px solid #ddd;
}

/* Active (expanded) nested lists */
#categoryTree .nested.active {
    display: block;
    animation: expand 0.3s ease-in-out;
}

/* Expand animation */
@keyframes expand {
    from {
        max-height: 0;
        opacity: 0;
    }
    to {
        max-height: 500px;
        opacity: 1;
    }
}

/* Professional hover effect for toggle button */
#categoryTree .toggle:hover {
    background-color: #007bff;
    color: #fff;
}
