# LifeDashboard
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Interactive Life Dashboard</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://d3js.org/d3.v7.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f8fafc; /* slate-50 */
        }

        #tree-container {
            overflow: auto; /* Enable scrolling for large trees */
        }

        /* --- D3 Tree Styles --- */
        .node {
            cursor: pointer;
        }

        .node circle {
            stroke-width: 3px;
        }

        .node text {
            font: 12px 'Inter', sans-serif;
            paint-order: stroke;
            stroke: #fff;
            stroke-width: 3px;
            stroke-linecap: butt;
            stroke-linejoin: miter;
        }
        
        .node .action-icon {
            fill: #94a3b8; /* slate-400 */
            stroke: #94a3b8;
            stroke-width: 0.5px;
            transition: fill 0.2s ease;
        }
        
        .node .action-icon:hover {
            fill: #1e293b; /* slate-800 */
        }

        .node .remove-icon:hover {
            fill: #ef4444; /* red-500 */
        }
        
        .node .log-icon {
            cursor: pointer;
        }

        .link {
            fill: none;
            stroke: #e2e8f0; /* slate-200 */
            stroke-width: 2px;
        }

        .node .center-circle {
            fill: #ffffff;
            stroke: #e2e8f0;
            stroke-width: 0.5px;
        }

        /* Spinner for AI analysis */
        .spinner {
            border: 4px solid rgba(0, 0, 0, 0.1);
            width: 36px;
            height: 36px;
            border-radius: 50%;
            border-left-color: #4f46e5; /* indigo-600 */
            animation: spin 1s ease infinite;
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        
        /* Calendar Styles */
        .calendar-day.logged-day {
            background-color: #dcfce7; /* green-100 */
            color: #166534; /* green-800 */
            font-weight: bold;
        }
        .calendar-day.today {
            border: 2px solid #4f46e5; /* indigo-600 */
        }
         .calendar-day.other-month {
            color: #9ca3af; /* gray-400 */
        }
        
        /* Drag and Drop styles */
        .dragging {
            opacity: 0.5;
            background: #f0f9ff; /* sky-50 */
        }
        .drag-over {
            border: 2px dashed #fb923c; /* orange-400 */
        }


    </style>
</head>
<body class="p-4 sm:p-6 md:p-8">

    <div class="max-w-full mx-auto bg-white p-6 rounded-xl shadow-lg">
        <div class="flex justify-between items-start mb-6 flex-wrap gap-4">
            <div class="flex items-center gap-4">
                <div class="text-left">
                     <h1 class="text-3xl font-bold text-gray-800">My Life Dashboard</h1>
                     <p id="current-date" class="text-lg font-semibold text-gray-500 mt-1"></p>
                </div>
                <button id="calendar-btn" class="p-2 text-gray-500 hover:text-indigo-600 hover:bg-gray-100 rounded-full transition-colors">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-7 w-7" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z" />
                    </svg>
                </button>
            </div>
            <div class="flex gap-2 flex-shrink-0 mt-1 flex-wrap">
                <button id="ai-creator-btn" class="px-4 py-2 bg-orange-500 text-white font-semibold rounded-lg shadow-md hover:bg-orange-600 focus:outline-none focus:ring-2 focus:ring-orange-400">AI Task Creator</button>
                <button id="ai-task-btn" class="px-4 py-2 bg-teal-500 text-white font-semibold rounded-lg shadow-md hover:bg-teal-600 focus:outline-none focus:ring-2 focus:ring-teal-400">AI Task Entry</button>
                <button id="ai-diary-btn" class="px-4 py-2 bg-purple-500 text-white font-semibold rounded-lg shadow-md hover:bg-purple-600 focus:outline-none focus:ring-2 focus:ring-purple-400">AI Diary Entry</button>
                <button id="add-pillar-btn" class="px-4 py-2 bg-indigo-500 text-white font-semibold rounded-lg shadow-md hover:bg-indigo-600 focus:outline-none focus:ring-2 focus:ring-indigo-400">Add Pillar</button>
                <button id="expand-all-btn" class="px-4 py-2 bg-sky-500 text-white font-semibold rounded-lg shadow-md hover:bg-sky-600 focus:outline-none focus:ring-2 focus:ring-sky-400">Expand All</button>
                <button id="collapse-all-btn" class="px-4 py-2 bg-slate-500 text-white font-semibold rounded-lg shadow-md hover:bg-slate-600 focus:outline-none focus:ring-2 focus:ring-slate-400">Collapse All</button>
                <input type="file" id="csv-import-input" class="hidden" accept=".csv">
                <button id="import-btn" class="px-4 py-2 bg-blue-500 text-white font-semibold rounded-lg shadow-md hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-400">Import CSV</button>
                <button id="export-btn" class="px-4 py-2 bg-green-500 text-white font-semibold rounded-lg shadow-md hover:bg-green-600 focus:outline-none focus:ring-2 focus:ring-green-400">Export to CSV</button>
                <button id="clear-all-btn" class="px-4 py-2 bg-red-500 text-white font-semibold rounded-lg shadow-md hover:bg-red-600 focus:outline-none focus:ring-2 focus:ring-red-400">Clear All</button>
            </div>
        </div>
        
        <div id="tree-container" class="tree-container border border-gray-200 rounded-lg min-h-[500px]"></div>

        <!-- Legend -->
        <div class="mt-8 pt-4 border-t border-gray-200">
            <h3 class="text-lg font-semibold text-gray-700 mb-3">Legend</h3>
            <div class="space-y-3">
                <div>
                    <h4 class="font-semibold text-gray-600">Node Border: Last Logged / Deadline</h4>
                    <p class="text-sm text-gray-500">
                        <b>Items:</b> The border color indicates the last time it was logged or its deadline status.
                        <br>
                        <b>Categories:</b> The border is a pie chart representing the status of its direct children.
                    </p>
                    <div class="flex flex-wrap gap-4 text-sm mt-1">
                        <div class="flex items-center gap-2"><div class="w-5 h-5 rounded-full border-2 border-green-500"></div><span>Logged Today / Deadline OK</span></div>
                        <div class="flex items-center gap-2"><div class="w-5 h-5 rounded-full border-2 border-yellow-400"></div><span>Logged > 1 Day / Deadline Approaching</span></div>
                        <div class="flex items-center gap-2"><div class="w-5 h-5 rounded-full border-2 border-orange-400"></div><span>Logged > 7 Days / Deadline Soon</span></div>
                        <div class="flex items-center gap-2"><div class="w-5 h-5 rounded-full border-2 border-red-500"></div><span>Logged > 30 Days / Deadline Overdue</span></div>
                        <div class="flex items-center gap-2"><div class="w-5 h-5 rounded-full border-2 border-gray-400"></div><span>No activity</span></div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Task View -->
    <div class="max-w-full mx-auto bg-white p-6 rounded-xl shadow-lg mt-8">
        <h2 class="text-2xl font-bold text-gray-800 mb-4">Prioritized Task List</h2>
        <div id="task-view-container" class="space-y-3">
            <!-- Tasks will be rendered here by JavaScript -->
        </div>
    </div>

    <!-- All Modals -->
    <div id="action-modal" class="fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full hidden z-50">
        <div class="relative top-20 mx-auto p-5 border w-full max-w-sm shadow-lg rounded-md bg-white">
            <h3 class="text-lg leading-6 font-medium text-gray-900" id="action-modal-title">Actions for...</h3>
            <div class="mt-4 flex flex-col gap-3">
                <button id="action-log-btn" class="w-full text-left px-4 py-2 bg-blue-500 text-white rounded-md hover:bg-blue-600">Log Activity</button>
                <button id="action-tasks-btn" class="w-full text-left px-4 py-2 bg-cyan-500 text-white rounded-md hover:bg-cyan-600">Manage Tasks</button>
                <button id="action-edit-btn" class="w-full text-left px-4 py-2 bg-gray-500 text-white rounded-md hover:bg-gray-600">Edit Name/Location</button>
                <button id="action-ai-creator-btn" class="w-full text-left px-4 py-2 bg-orange-500 text-white rounded-md hover:bg-orange-600">AI Create Children</button>
                <button id="action-goal-btn" class="w-full text-left px-4 py-2 bg-yellow-500 text-white rounded-md hover:bg-yellow-600">Set Goal</button>
                <button id="action-history-btn" class="w-full text-left px-4 py-2 bg-gray-500 text-white rounded-md hover:bg-gray-600">View History</button>
                <button id="action-notes-btn" class="w-full text-left px-4 py-2 bg-gray-500 text-white rounded-md hover:bg-gray-600">View/Add Notes</button>
                <button id="action-add-btn" class="w-full text-left px-4 py-2 bg-green-500 text-white rounded-md hover:bg-green-600">Add Child Item</button>
                <button id="action-delete-btn" class="w-full text-left px-4 py-2 bg-red-500 text-white rounded-md hover:bg-red-600">Delete Item</button>
            </div>
            <div class="items-center px-4 py-3 -mx-5 -mb-5 mt-4 bg-gray-50 rounded-b-md flex justify-end">
                <button id="close-action-modal-btn" class="px-4 py-2 bg-gray-200 text-gray-800 text-base font-medium rounded-md shadow-sm hover:bg-gray-300">Close</button>
            </div>
        </div>
    </div>

    <div id="log-modal" class="fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full hidden z-50">
        <div class="relative top-20 mx-auto p-5 border w-full max-w-lg shadow-lg rounded-md bg-white">
            <h3 class="text-lg leading-6 font-medium text-gray-900" id="log-modal-title">Log Activity</h3>
            <p class="text-sm text-gray-500 mt-1" id="log-modal-subtitle"></p>
            <div class="mt-4"><textarea id="log-note-input" class="w-full h-24 p-2 border border-gray-300 rounded-md" placeholder="Add an optional note..."></textarea></div>
            <div class="items-center px-4 py-3 -mx-5 -mb-5 mt-4 bg-gray-50 rounded-b-md flex justify-end gap-2">
                <button id="confirm-log" class="px-4 py-2 bg-blue-500 text-white text-base font-medium rounded-md shadow-sm hover:bg-blue-600">Log it!</button>
                <button id="cancel-log" class="mt-2 px-4 py-2 bg-gray-200 text-gray-800 text-base font-medium rounded-md shadow-sm hover:bg-gray-300">Cancel</button>
            </div>
        </div>
    </div>
    <div id="notes-modal" class="fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full hidden z-50">
        <div class="relative top-10 mx-auto p-5 border w-full max-w-lg shadow-lg rounded-md bg-white">
             <h3 class="text-lg leading-6 font-medium text-gray-900" id="notes-modal-title">Notes for...</h3>
             <div id="notes-list" class="mt-4 space-y-3 max-h-60 overflow-y-auto pr-2"></div>
             <div class="mt-4"><textarea id="note-input" class="w-full h-24 p-2 border border-gray-300 rounded-md" placeholder="Add a new note..."></textarea></div>
             <div class="items-center px-4 py-3 -mx-5 -mb-5 mt-4 bg-gray-50 rounded-b-md flex justify-end gap-2">
                 <button id="save-note-btn" class="px-4 py-2 bg-blue-500 text-white text-base font-medium rounded-md shadow-sm hover:bg-blue-600">Add Note</button>
                 <button id="close-notes-modal-btn" class="px-4 py-2 bg-gray-200 text-gray-800 text-base font-medium rounded-md shadow-sm hover:bg-gray-300">Close</button>
             </div>
        </div>
    </div>
    <div id="history-modal" class="fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full hidden z-50">
        <div class="relative top-10 mx-auto p-5 border w-full max-w-lg shadow-lg rounded-md bg-white">
             <h3 class="text-lg leading-6 font-medium text-gray-900" id="history-modal-title">Log History for...</h3>
             <div id="history-list" class="mt-4 space-y-3 max-h-80 overflow-y-auto pr-2"></div>
             <div class="items-center px-4 py-3 -mx-5 -mb-5 mt-4 bg-gray-50 rounded-b-md flex justify-end">
                 <button id="close-history-modal-btn" class="px-4 py-2 bg-gray-200 text-gray-800 text-base font-medium rounded-md shadow-sm hover:bg-gray-300">Close</button>
             </div>
        </div>
    </div>
    <div id="goal-modal" class="fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full hidden z-50">
        <div class="relative top-10 mx-auto p-5 border w-full max-w-lg shadow-lg rounded-md bg-white">
             <h3 class="text-lg leading-6 font-medium text-gray-900" id="goal-modal-title">Set Goal for...</h3>
             <div class="mt-4">
                 <input type="text" id="goal-text-input" class="w-full p-2 border border-gray-300 rounded-md" placeholder="Enter goal description...">
                 <input type="date" id="goal-date-input" class="w-full p-2 border border-gray-300 rounded-md mt-2">
             </div>
             <div id="goal-list" class="mt-4 space-y-3 max-h-60 overflow-y-auto pr-2"></div>
             <div class="items-center px-4 py-3 -mx-5 -mb-5 mt-4 bg-gray-50 rounded-b-md flex justify-end gap-2">
                 <button id="save-goal-btn" class="px-4 py-2 bg-yellow-500 text-white text-base font-medium rounded-md shadow-sm hover:bg-yellow-600">Set Goal</button>
                 <button id="close-goal-modal-btn" class="px-4 py-2 bg-gray-200 text-gray-800 text-base font-medium rounded-md shadow-sm hover:bg-gray-300">Close</button>
             </div>
        </div>
    </div>
    <div id="add-modal" class="fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full hidden z-50">
        <div class="relative top-20 mx-auto p-5 border w-full max-w-lg shadow-lg rounded-md bg-white">
            <h3 class="text-lg leading-6 font-medium text-gray-900" id="add-modal-title">Add New Item</h3>
            <div class="mt-4"><input type="text" id="add-item-input" class="w-full p-2 border border-gray-300 rounded-md" placeholder="Name of new item..."></div>
            <div class="items-center px-4 py-3 -mx-5 -mb-5 mt-4 bg-gray-50 rounded-b-md flex justify-end gap-2">
                <button id="confirm-add-btn" class="px-4 py-2 bg-blue-500 text-white text-base font-medium rounded-md shadow-sm hover:bg-blue-600">Add Item</button>
                <button id="cancel-add-btn" class="mt-2 px-4 py-2 bg-gray-200 text-gray-800 text-base font-medium rounded-md shadow-sm hover:bg-gray-300">Cancel</button>
            </div>
        </div>
    </div>
     <div id="delete-modal" class="fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full hidden z-50">
        <div class="relative top-20 mx-auto p-5 border w-full max-w-lg shadow-lg rounded-md bg-white">
            <h3 class="text-lg leading-6 font-medium text-red-600" id="delete-modal-title">Confirm Deletion</h3>
            <p class="mt-2 text-sm text-gray-600" id="delete-modal-text"></p>
            <p class="mt-2 text-sm font-bold text-red-500">This action is permanent and cannot be undone.</p>
            <div class="items-center px-4 py-3 -mx-5 -mb-5 mt-4 bg-gray-50 rounded-b-md flex justify-end gap-2">
                <button id="confirm-delete-btn" class="px-4 py-2 bg-red-600 text-white text-base font-medium rounded-md shadow-sm hover:bg-red-700">Delete</button>
                <button id="cancel-delete-btn" class="mt-2 px-4 py-2 bg-gray-200 text-gray-800 text-base font-medium rounded-md shadow-sm hover:bg-gray-300">Cancel</button>
            </div>
        </div>
    </div>
    
    <!-- AI Diary Modal -->
    <div id="ai-diary-modal" class="fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full hidden z-50">
        <div class="relative top-10 mx-auto p-5 border w-full max-w-2xl shadow-lg rounded-md bg-white">
            <h3 class="text-xl leading-6 font-bold text-gray-900">AI-Powered Diary Entry</h3>
            <p class="text-sm text-gray-500 mt-1">Write about your day, and the AI will suggest where to log it.</p>
            
            <div class="mt-4">
                <textarea id="ai-diary-input" class="w-full h-48 p-3 border border-gray-300 rounded-md" placeholder="Today I went to the gym and then had a great call with my dad..."></textarea>
            </div>

            <div id="ai-analysis-section" class="mt-4">
                <div id="ai-spinner" class="hidden justify-center items-center flex-col gap-2">
                    <div class="spinner"></div>
                    <p class="text-gray-600">Analyzing your entry...</p>
                </div>
                <div id="ai-suggestions-container" class="hidden">
                    <h4 class="font-semibold text-gray-700">AI Suggestions:</h4>
                    <p class="text-sm text-gray-500">Review and confirm the categories to log this entry against.</p>
                    <div id="ai-suggestions-list" class="mt-2 space-y-2 max-h-48 overflow-y-auto border border-gray-200 p-3 rounded-md"></div>
                </div>
                 <div id="ai-error-message" class="hidden text-red-600 bg-red-100 p-3 rounded-md"></div>
            </div>

            <div class="items-center px-4 py-3 -mx-5 -mb-5 mt-4 bg-gray-50 rounded-b-md flex justify-end gap-2">
                <button id="analyze-diary-btn" class="px-4 py-2 bg-purple-500 text-white text-base font-medium rounded-md shadow-sm hover:bg-purple-600">Analyze Entry</button>
                <button id="confirm-ai-logs-btn" class="hidden px-4 py-2 bg-green-500 text-white text-base font-medium rounded-md shadow-sm hover:bg-green-600">Confirm Logs</button>
                <button id="cancel-ai-diary-btn" class="px-4 py-2 bg-gray-200 text-gray-800 text-base font-medium rounded-md shadow-sm hover:bg-gray-300">Cancel</button>
            </div>
        </div>
    </div>

    <!-- AI Task Modal -->
    <div id="ai-task-modal" class="fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full hidden z-50">
        <div class="relative top-10 mx-auto p-5 border w-full max-w-2xl shadow-lg rounded-md bg-white">
            <h3 class="text-xl leading-6 font-bold text-gray-900">AI-Powered Task Entry</h3>
            <p class="text-sm text-gray-500 mt-1">Describe your tasks, and the AI will categorize them for you.</p>
            
            <div class="mt-4">
                <textarea id="ai-task-input" class="w-full h-48 p-3 border border-gray-300 rounded-md" placeholder="Book flight to Melbourne for next Friday and prepare presentation for the Wednesday meeting..."></textarea>
            </div>

            <div id="ai-task-analysis-section" class="mt-4">
                <div id="ai-task-spinner" class="hidden justify-center items-center flex-col gap-2">
                    <div class="spinner"></div>
                    <p class="text-gray-600">Analyzing your tasks...</p>
                </div>
                <div id="ai-task-suggestions-container" class="hidden">
                    <h4 class="font-semibold text-gray-700">AI Suggestions:</h4>
                    <p class="text-sm text-gray-500">Review and confirm the tasks and categories below.</p>
                    <div id="ai-task-suggestions-list" class="mt-2 space-y-2 max-h-48 overflow-y-auto border border-gray-200 p-3 rounded-md"></div>
                </div>
                 <div id="ai-task-error-message" class="hidden text-red-600 bg-red-100 p-3 rounded-md"></div>
            </div>

            <div class="items-center px-4 py-3 -mx-5 -mb-5 mt-4 bg-gray-50 rounded-b-md flex justify-end gap-2">
                <button id="analyze-task-btn" class="px-4 py-2 bg-teal-500 text-white text-base font-medium rounded-md shadow-sm hover:bg-teal-600">Analyze Tasks</button>
                <button id="confirm-ai-tasks-btn" class="hidden px-4 py-2 bg-green-500 text-white text-base font-medium rounded-md shadow-sm hover:bg-green-600">Confirm Tasks</button>
                <button id="cancel-ai-task-btn" class="px-4 py-2 bg-gray-200 text-gray-800 text-base font-medium rounded-md shadow-sm hover:bg-gray-300">Cancel</button>
            </div>
        </div>
    </div>

    <!-- AI Creator Modal -->
    <div id="ai-creator-modal" class="fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full hidden z-50">
        <div class="relative top-10 mx-auto p-5 border w-full max-w-2xl shadow-lg rounded-md bg-white">
            <h3 class="text-xl leading-6 font-bold text-gray-900">AI Task Creator</h3>
            <p class="text-sm text-gray-500 mt-1">Describe a new project or goal, and the AI will create a new node for it.</p>
            
            <div class="mt-4">
                <textarea id="ai-creator-input" class="w-full h-48 p-3 border border-gray-300 rounded-md" placeholder="Plan a trip to Japan..."></textarea>
            </div>

            <div id="ai-creator-analysis-section" class="mt-4">
                <div id="ai-creator-spinner" class="hidden justify-center items-center flex-col gap-2">
                    <div class="spinner"></div>
                    <p class="text-gray-600">Analyzing...</p>
                </div>
                <div id="ai-creator-suggestions-container" class="hidden">
                    <h4 class="font-semibold text-gray-700">AI Suggestion:</h4>
                    <div id="ai-creator-suggestions-list" class="mt-2 space-y-2 border border-gray-200 p-3 rounded-md"></div>
                </div>
                 <div id="ai-creator-error-message" class="hidden text-red-600 bg-red-100 p-3 rounded-md"></div>
            </div>

            <div class="items-center px-4 py-3 -mx-5 -mb-5 mt-4 bg-gray-50 rounded-b-md flex justify-end gap-2">
                <button id="analyze-creator-btn" class="px-4 py-2 bg-orange-500 text-white text-base font-medium rounded-md shadow-sm hover:bg-orange-600">Analyze</button>
                <button id="confirm-ai-creator-btn" class="hidden px-4 py-2 bg-green-500 text-white text-base font-medium rounded-md shadow-sm hover:bg-green-600">Confirm Creation</button>
                <button id="cancel-ai-creator-btn" class="px-4 py-2 bg-gray-200 text-gray-800 text-base font-medium rounded-md shadow-sm hover:bg-gray-300">Cancel</button>
            </div>
        </div>
    </div>
    
    <!-- Import Confirmation Modal -->
    <div id="import-confirm-modal" class="fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full hidden z-50">
        <div class="relative top-20 mx-auto p-5 border w-full max-w-lg shadow-lg rounded-md bg-white">
            <h3 class="text-lg leading-6 font-medium text-yellow-600">Confirm Import</h3>
            <p class="mt-2 text-sm text-gray-600">Are you sure you want to import this file? This will overwrite your entire current dashboard.</p>
            <p class="mt-2 text-sm font-bold text-red-500">This action cannot be undone.</p>
            <div class="items-center px-4 py-3 -mx-5 -mb-5 mt-4 bg-gray-50 rounded-b-md flex justify-end gap-2">
                <button id="confirm-import-btn" class="px-4 py-2 bg-blue-600 text-white text-base font-medium rounded-md shadow-sm hover:bg-blue-700">Yes, Import</button>
                <button id="cancel-import-btn" class="mt-2 px-4 py-2 bg-gray-200 text-gray-800 text-base font-medium rounded-md shadow-sm hover:bg-gray-300">Cancel</button>
            </div>
        </div>
    </div>

    <!-- Export Filename Modal -->
    <div id="export-modal" class="fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full hidden z-50">
        <div class="relative top-20 mx-auto p-5 border w-full max-w-lg shadow-lg rounded-md bg-white">
            <h3 class="text-lg leading-6 font-medium text-gray-900">Export Dashboard</h3>
            <div class="mt-4">
                <label for="export-filename-input" class="block text-sm font-medium text-gray-700">Filename</label>
                <input type="text" id="export-filename-input" class="w-full p-2 border border-gray-300 rounded-md mt-1" placeholder="life-dashboard-export">
            </div>
            <div class="items-center px-4 py-3 -mx-5 -mb-5 mt-4 bg-gray-50 rounded-b-md flex justify-end gap-2">
                <button id="confirm-export-btn" class="px-4 py-2 bg-green-600 text-white text-base font-medium rounded-md shadow-sm hover:bg-green-700">Export</button>
                <button id="cancel-export-btn" class="mt-2 px-4 py-2 bg-gray-200 text-gray-800 text-base font-medium rounded-md shadow-sm hover:bg-gray-300">Cancel</button>
            </div>
        </div>
    </div>
    
    <!-- Task Modal -->
    <div id="task-modal" class="fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full hidden z-50">
        <div class="relative top-10 mx-auto p-5 border w-full max-w-xl shadow-lg rounded-md bg-white">
             <h3 class="text-lg leading-6 font-medium text-gray-900" id="task-modal-title">Manage Tasks for...</h3>
             <div class="mt-4">
                <h4 class="text-md font-medium text-gray-800">Tasks</h4>
                <div id="task-list" class="mt-2 space-y-2 max-h-60 overflow-y-auto pr-2 border-t pt-2"></div>
                <div class="mt-2 flex gap-2 items-center">
                    <input type="text" id="task-input" class="w-full p-2 border border-gray-300 rounded-md" placeholder="Add a new task...">
                    <input type="date" id="task-deadline-input" class="p-2 border border-gray-300 rounded-md">
                    <button id="add-task-btn" class="px-4 py-2 bg-cyan-500 text-white font-semibold rounded-lg shadow-md hover:bg-cyan-600">Add</button>
                </div>
             </div>
             <div class="items-center px-4 py-3 -mx-5 -mb-5 mt-4 bg-gray-50 rounded-b-md flex justify-end gap-2">
                 <button id="save-tasks-btn" class="px-4 py-2 bg-blue-500 text-white text-base font-medium rounded-md shadow-sm hover:bg-blue-600">Save Changes</button>
                 <button id="close-task-modal-btn" class="px-4 py-2 bg-gray-200 text-gray-800 text-base font-medium rounded-md shadow-sm hover:bg-gray-300">Close</button>
             </div>
        </div>
    </div>

    <!-- Clear All Confirmation Modal -->
    <div id="clear-all-modal" class="fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full hidden z-50">
        <div class="relative top-20 mx-auto p-5 border w-full max-w-lg shadow-lg rounded-md bg-white">
            <h3 class="text-lg leading-6 font-medium text-red-600">Confirm Clear All</h3>
            <p class="mt-2 text-sm text-gray-600">Are you sure you want to delete all pillars and tasks? This will reset the entire dashboard.</p>
            <p class="mt-2 text-sm font-bold text-red-500">This action is permanent and cannot be undone.</p>
            <div class="items-center px-4 py-3 -mx-5 -mb-5 mt-4 bg-gray-50 rounded-b-md flex justify-end gap-2">
                <button id="confirm-clear-all-btn" class="px-4 py-2 bg-red-600 text-white text-base font-medium rounded-md shadow-sm hover:bg-red-700">Yes, Clear Everything</button>
                <button id="cancel-clear-all-btn" class="mt-2 px-4 py-2 bg-gray-200 text-gray-800 text-base font-medium rounded-md shadow-sm hover:bg-gray-300">Cancel</button>
            </div>
        </div>
    </div>

    <!-- Calendar Modal -->
    <div id="calendar-modal" class="fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full hidden z-50">
        <div class="relative top-20 mx-auto p-5 border w-full max-w-md shadow-lg rounded-md bg-white">
            <div class="flex justify-between items-center mb-4">
                <h3 class="text-lg leading-6 font-medium text-gray-900" id="calendar-month-year"></h3>
                <div class="flex gap-2">
                    <button id="prev-month-btn" class="p-1 rounded-full hover:bg-gray-200">&lt;</button>
                    <button id="next-month-btn" class="p-1 rounded-full hover:bg-gray-200">&gt;</button>
                </div>
            </div>
            <div class="text-center mb-4">
                <span class="text-lg font-semibold">Streak: 🔥 <span id="streak-counter">0</span> days</span>
            </div>
            <div id="calendar-grid" class="grid grid-cols-7 gap-1 text-center">
                <!-- Calendar will be rendered here -->
            </div>
            <div class="items-center px-4 py-3 -mx-5 -mb-5 mt-4 bg-gray-50 rounded-b-md flex justify-end">
                <button id="close-calendar-modal-btn" class="px-4 py-2 bg-gray-200 text-gray-800 text-base font-medium rounded-md shadow-sm hover:bg-gray-300">Close</button>
            </div>
        </div>
    </div>
    
    <!-- Edit Modal -->
    <div id="edit-modal" class="fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full hidden z-50">
        <div class="relative top-20 mx-auto p-5 border w-full max-w-lg shadow-lg rounded-md bg-white">
            <h3 class="text-lg leading-6 font-medium text-gray-900">Edit Node</h3>
            <div class="mt-4">
                <label for="edit-name-input" class="block text-sm font-medium text-gray-700">Node Name</label>
                <input type="text" id="edit-name-input" class="w-full p-2 border border-gray-300 rounded-md mt-1">
            </div>
            <div class="mt-4">
                <label for="edit-path-input" class="block text-sm font-medium text-gray-700">Parent Path (e.g., Hobbies/Music)</label>
                <input type="text" id="edit-path-input" class="w-full p-2 border border-gray-300 rounded-md mt-1">
            </div>
            <div class="items-center px-4 py-3 -mx-5 -mb-5 mt-4 bg-gray-50 rounded-b-md flex justify-end gap-2">
                <button id="confirm-edit-btn" class="px-4 py-2 bg-blue-500 text-white text-base font-medium rounded-md shadow-sm hover:bg-blue-600">Save Changes</button>
                <button id="cancel-edit-btn" class="mt-2 px-4 py-2 bg-gray-200 text-gray-800 text-base font-medium rounded-md shadow-sm hover:bg-gray-300">Cancel</button>
            </div>
        </div>
    </div>


    <!-- Save Feedback Toast -->
    <div id="save-feedback" class="fixed bottom-5 right-5 bg-green-600 text-white px-5 py-3 rounded-lg shadow-lg opacity-0 transition-opacity duration-500 pointer-events-none">
        Saved!
    </div>


    <script>
        // --- DATA STRUCTURE ---
        let lifePillars = []; // Start empty, will be populated by loadData
        let dailyLogs = {}; // To track daily interactions

        // Default structure if no saved data is found
        const defaultLifePillars = [
            { name: "Relationship", children: [ { name: "Mia", children: [] }, { name: "Family", children: [ { name: "Mum", children: [] }, { name: "Dad", children: [] }, { name: "Marlon", children: [] }, { name: "Flynn", children: [] } ] }, { name: "Extended Family", children: [ { name: "Grandparents", children: [] }, { name: "Cousins", children: [] } ] } ] }, { name: "Friendships", children: [ { name: "Tully", children: [] }, { name: "Kye", children: [] }, { name: "Jcb", children: [] }, { name: "Swilk", children: [] }, { name: "Jez", children: [] }, { name: "Horgs", children: [] }, { name: "Damo", children: [] } ] }, { name: "Uni", children: [ { name: "Research Positions", children: [] }, { name: "Current Courses", children: [ { name: "Statistics", children: [] }, { name: "Proteomics", children: [] }, { name: "Medical Microbiology", children: [] }, { name: "Infectious Diseases", children: [] } ] } ] },
            { name: "Career", children: [ { name: "CV", children: [] }, { name: "Goal", children: [] }, { name: "Current Job", children: [] }, { name: "Seek", children: [] } ] }, { name: "Health", children: [ { name: "Stretching", children: [] }, { name: "Mental", children: [] }, { name: "Gym", children: [] }, { name: "ITB", children: [] }, { name: "Lungs", children: [] } ] }, { name: "Finance", children: [ { name: "Stocks", children: [] }, { name: "Savings", children: [] }, { name: "Income", children: [] }, { name: "Spending", children: [] }, { name: "Wants", children: [] }, { name: "Needs", children: [] }, { name: "Taxes", children: [] } ] },
            { name: "Hobbies", children: [] }, { name: "Tasks", children: [] }, { name: "Car", children: [] },
        ];

        // --- DOM ELEMENTS ---
        const treeContainer = document.getElementById('tree-container');
        const actionModal = { el: document.getElementById('action-modal'), title: document.getElementById('action-modal-title'), logBtn: document.getElementById('action-log-btn'), tasksBtn: document.getElementById('action-tasks-btn'), editBtn: document.getElementById('action-edit-btn'), aiCreatorBtn: document.getElementById('action-ai-creator-btn'), goalBtn: document.getElementById('action-goal-btn'), historyBtn: document.getElementById('action-history-btn'), notesBtn: document.getElementById('action-notes-btn'), addBtn: document.getElementById('action-add-btn'), deleteBtn: document.getElementById('action-delete-btn'), closeBtn: document.getElementById('close-action-modal-btn') };
        const logModal = { el: document.getElementById('log-modal'), subtitle: document.getElementById('log-modal-subtitle'), input: document.getElementById('log-note-input'), confirmBtn: document.getElementById('confirm-log'), cancelBtn: document.getElementById('cancel-log') };
        const notesModal = { el: document.getElementById('notes-modal'), title: document.getElementById('notes-modal-title'), list: document.getElementById('notes-list'), input: document.getElementById('note-input'), saveBtn: document.getElementById('save-note-btn'), closeBtn: document.getElementById('close-notes-modal-btn') };
        const historyModal = { el: document.getElementById('history-modal'), title: document.getElementById('history-modal-title'), list: document.getElementById('history-list'), closeBtn: document.getElementById('close-history-modal-btn') };
        const goalModal = { el: document.getElementById('goal-modal'), title: document.getElementById('goal-modal-title'), textInput: document.getElementById('goal-text-input'), dateInput: document.getElementById('goal-date-input'), list: document.getElementById('goal-list'), saveBtn: document.getElementById('save-goal-btn'), closeBtn: document.getElementById('close-goal-modal-btn') };
        const addModal = { el: document.getElementById('add-modal'), title: document.getElementById('add-modal-title'), input: document.getElementById('add-item-input'), confirmBtn: document.getElementById('confirm-add-btn'), cancelBtn: document.getElementById('cancel-add-btn') };
        const deleteModal = { el: document.getElementById('delete-modal'), title: document.getElementById('delete-modal-title'), text: document.getElementById('delete-modal-text'), confirmBtn: document.getElementById('confirm-delete-btn'), cancelBtn: document.getElementById('cancel-delete-btn') };
        const aiDiaryModal = { el: document.getElementById('ai-diary-modal'), input: document.getElementById('ai-diary-input'), analyzeBtn: document.getElementById('analyze-diary-btn'), confirmBtn: document.getElementById('confirm-ai-logs-btn'), cancelBtn: document.getElementById('cancel-ai-diary-btn'), spinner: document.getElementById('ai-spinner'), suggestionsContainer: document.getElementById('ai-suggestions-container'), suggestionsList: document.getElementById('ai-suggestions-list'), errorMessage: document.getElementById('ai-error-message') };
        const aiTaskModal = { el: document.getElementById('ai-task-modal'), input: document.getElementById('ai-task-input'), analyzeBtn: document.getElementById('analyze-task-btn'), confirmBtn: document.getElementById('confirm-ai-tasks-btn'), cancelBtn: document.getElementById('cancel-ai-task-btn'), spinner: document.getElementById('ai-task-spinner'), suggestionsContainer: document.getElementById('ai-task-suggestions-container'), suggestionsList: document.getElementById('ai-task-suggestions-list'), errorMessage: document.getElementById('ai-task-error-message') };
        const aiCreatorModal = { el: document.getElementById('ai-creator-modal'), input: document.getElementById('ai-creator-input'), analyzeBtn: document.getElementById('analyze-creator-btn'), confirmBtn: document.getElementById('confirm-ai-creator-btn'), cancelBtn: document.getElementById('cancel-ai-creator-btn'), spinner: document.getElementById('ai-creator-spinner'), suggestionsContainer: document.getElementById('ai-creator-suggestions-container'), suggestionsList: document.getElementById('ai-creator-suggestions-list'), errorMessage: document.getElementById('ai-creator-error-message') };
        const importConfirmModal = { el: document.getElementById('import-confirm-modal'), confirmBtn: document.getElementById('confirm-import-btn'), cancelBtn: document.getElementById('cancel-import-btn') };
        const exportModal = { el: document.getElementById('export-modal'), input: document.getElementById('export-filename-input'), confirmBtn: document.getElementById('confirm-export-btn'), cancelBtn: document.getElementById('cancel-export-btn') };
        const taskModal = { el: document.getElementById('task-modal'), title: document.getElementById('task-modal-title'), taskList: document.getElementById('task-list'), taskInput: document.getElementById('task-input'), deadlineInput: document.getElementById('task-deadline-input'), addBtn: document.getElementById('add-task-btn'), saveBtn: document.getElementById('save-tasks-btn'), closeBtn: document.getElementById('close-task-modal-btn') };
        const clearAllModal = { el: document.getElementById('clear-all-modal'), confirmBtn: document.getElementById('confirm-clear-all-btn'), cancelBtn: document.getElementById('cancel-clear-all-btn') };
        const calendarModal = { el: document.getElementById('calendar-modal'), monthYear: document.getElementById('calendar-month-year'), grid: document.getElementById('calendar-grid'), prevBtn: document.getElementById('prev-month-btn'), nextBtn: document.getElementById('next-month-btn'), closeBtn: document.getElementById('close-calendar-modal-btn'), streakCounter: document.getElementById('streak-counter') };
        const editModal = { el: document.getElementById('edit-modal'), nameInput: document.getElementById('edit-name-input'), pathInput: document.getElementById('edit-path-input'), confirmBtn: document.getElementById('confirm-edit-btn'), cancelBtn: document.getElementById('cancel-edit-btn') };
        const importBtn = document.getElementById('import-btn');
        const importInput = document.getElementById('csv-import-input');
        const exportBtn = document.getElementById('export-btn');
        const addPillarBtn = document.getElementById('add-pillar-btn');
        const expandAllBtn = document.getElementById('expand-all-btn');
        const collapseAllBtn = document.getElementById('collapse-all-btn');
        const aiDiaryBtn = document.getElementById('ai-diary-btn');
        const aiTaskBtn = document.getElementById('ai-task-btn');
        const aiCreatorBtn = document.getElementById('ai-creator-btn');
        const clearAllBtn = document.getElementById('clear-all-btn');
        const calendarBtn = document.getElementById('calendar-btn');

        let nodeToLog = null, nodeForNotes = null, nodeForHistory = null, nodeForGoal = null, nodeToAddTo = null, nodeForTasks = null, nodeToEdit = null;
        let itemToDelete = { item: null, type: null, parent: null };
        let activeNodeForActions = null;
        let fileToImport = null;
        let currentCalendarDate = new Date();
        let aiCreatorContextNode = null;
        
        // --- DATA MANAGEMENT ---
        function initializeNodeProperties(nodes) {
            nodes.forEach(node => {
                if (!('notes' in node)) node.notes = [];
                if (!('logs' in node)) node.logs = [];
                if (!('goals' in node)) node.goals = [];
                if (!('tasks' in node)) node.tasks = [];
                if (!('createdAt' in node)) node.createdAt = new Date().toISOString();
                if (!('children' in node)) node.children = [];
                if (node.children.length > 0) initializeNodeProperties(node.children);
            });
        }

        function loadData() {
            const savedPillars = localStorage.getItem('lifePillarsData');
            if (savedPillars) {
                try { lifePillars = JSON.parse(savedPillars); } 
                catch (e) { console.error("Failed to parse saved pillar data, using default.", e); lifePillars = defaultLifePillars; }
            } else { lifePillars = defaultLifePillars; }
            initializeNodeProperties(lifePillars);

            const savedLogs = localStorage.getItem('dailyLogs');
            if(savedLogs) {
                try { dailyLogs = JSON.parse(savedLogs); }
                catch(e) { console.error("Failed to parse daily logs.", e); dailyLogs = {}; }
            }
        }

        function saveData() {
            try {
                localStorage.setItem('lifePillarsData', JSON.stringify(lifePillars));
                localStorage.setItem('dailyLogs', JSON.stringify(dailyLogs));
                const feedbackEl = document.getElementById('save-feedback');
                if (feedbackEl) {
                    feedbackEl.classList.remove('opacity-0');
                    setTimeout(() => { feedbackEl.classList.add('opacity-0'); }, 1500);
                }
            } catch (e) { console.error("Failed to save data:", e); }
        }

        function logInteractionForToday() {
            const today = new Date();
            const dateString = `${today.getFullYear()}-${String(today.getMonth() + 1).padStart(2, '0')}-${String(today.getDate()).padStart(2, '0')}`;
            dailyLogs[dateString] = true;
            saveData();
        }

        // --- COLOR LOGIC ---
        function getBorderColor(nodeData) {
            if (nodeData.name === "root") return '#334155'; // slate-700

            const incompleteTasks = nodeData.tasks ? nodeData.tasks.filter(t => !t.completed && t.deadline) : [];

            if (incompleteTasks.length > 0) {
                const now = new Date();
                let mostUrgentDays = Infinity;

                incompleteTasks.forEach(task => {
                    const deadline = new Date(task.deadline);
                    const timeRemaining = deadline - now;
                    const daysRemaining = timeRemaining / (1000 * 60 * 60 * 24);
                    if (daysRemaining < mostUrgentDays) {
                        mostUrgentDays = daysRemaining;
                    }
                });

                if (mostUrgentDays < 0) return '#ef4444'; // Overdue
                if (mostUrgentDays <= 2) return '#ef4444'; 
                if (mostUrgentDays <= 7) return '#fb923c'; 
                if (mostUrgentDays <= 30) return '#facc15';
                
                return '#22c55e';
            }

            // Fallback to log-based color
            const allLogs = [];
            d3.hierarchy(nodeData).each(d => {
                if (d.data.logs) {
                    allLogs.push(...d.data.logs);
                }
            });

            if (allLogs.length === 0) return '#cbd5e1'; // gray-300
            
            const mostRecentLog = allLogs.reduce((latest, current) => new Date(latest.timestamp) > new Date(current.timestamp) ? latest : current);
            const diffDays = (new Date() - new Date(mostRecentLog.timestamp)) / (1000 * 60 * 60 * 24);

            if (diffDays <= 1) return '#22c55e'; // green-500
            if (diffDays <= 7) return '#facc15'; // yellow-400
            if (diffDays <= 30) return '#fb923c'; // orange-400
            return '#ef4444'; // red-500
        }

        function getNodeBorderData(d3Node) {
            const nodeData = d3Node.data;
            const children = d3Node.children || d3Node._children;

            if (!children || children.length === 0) {
                return [{
                    startAngle: 0,
                    endAngle: 2 * Math.PI,
                    color: getBorderColor(nodeData)
                }];
            }

            const totalChildren = children.length;
            const colorCategories = { '#22c55e': 0, '#facc15': 0, '#fb923c': 0, '#ef4444': 0, '#cbd5e1': 0 };
            children.forEach(c => {
                const color = getBorderColor(c.data);
                if (color in colorCategories) colorCategories[color]++;
            });

            const pieData = [];
            let currentAngle = 0;
            const colorOrder = ['#22c55e', '#facc15', '#fb923c', '#ef4444', '#cbd5e1'];
            colorOrder.forEach(color => {
                const count = colorCategories[color];
                if (count > 0) {
                    const angle = (count / totalChildren) * 2 * Math.PI;
                    pieData.push({ startAngle: currentAngle, endAngle: currentAngle + angle, color: color });
                    currentAngle += angle;
                }
            });

            if (pieData.length === 0) {
                 return [{ startAngle: 0, endAngle: 2 * Math.PI, color: '#cbd5e1' }];
            }
            return pieData;
        }


        // --- D3 RENDERING & VISUALS ---
        let d3Root, svg, tree, i = 0, duration = 750;

        function renderD3Tree() {
            treeContainer.innerHTML = '';
            const margin = {top: 40, right: 120, bottom: 40, left: 120};
            const initialWidth = treeContainer.clientWidth - margin.left - margin.right;
            const initialHeight = treeContainer.clientHeight - margin.top - margin.bottom;
            tree = d3.tree().size([initialHeight, initialWidth]);
            svg = d3.select("#tree-container").append("svg")
                .attr("width", initialWidth + margin.right + margin.left)
                .attr("height", initialHeight + margin.top + margin.bottom)
              .append("g")
                .attr("transform", "translate(" + margin.left + "," + margin.top + ")");
            
            const data = { name: "root", children: lifePillars };
            d3Root = d3.hierarchy(data, d => d.children);
            d3Root.x0 = initialHeight / 2;
            d3Root.y0 = 0;
            
            if (d3Root.children) {
                d3Root.children.forEach(collapse);
            }
            
            updateD3(d3Root);
        }

        function collapse(d) {
            if (d.children) {
                d._children = d.children;
                d._children.forEach(collapse);
                d.children = null;
            }
        }

        function updateD3(source) {
            const treeData = tree(d3Root);
            const nodes = treeData.descendants();
            const links = treeData.descendants().slice(1);
            
            let maxY = 0;
            nodes.forEach(d => {
                d.y = d.depth * 220;
                if (d.y > maxY) maxY = d.y;
            });

            const newWidth = maxY + 300;
            d3.select('#tree-container svg').attr('width', newWidth);
            
            let minX = Infinity, maxX = -Infinity;
            nodes.forEach(d => {
                if(d.x < minX) minX = d.x;
                if(d.x > maxX) maxX = d.x;
            });
            const newHeight = maxX - minX + 80;
            d3.select('#tree-container svg').attr('height', Math.max(newHeight, treeContainer.clientHeight));
            svg.attr("transform", "translate(" + 120 + "," + (Math.abs(minX) + 40) + ")");


            const node = svg.selectAll('g.node').data(nodes, d => d.id || (d.id = ++i));
            const nodeEnter = node.enter().append('g')
                .attr('class', 'node')
                .attr("transform", d => "translate(" + source.y0 + "," + source.x0 + ")");

            const radius = 10;
            const borderWidth = 3;

            const donutContainer = nodeEnter.append('g')
                .attr('class', 'donut-container')
                .on('click', toggleChildren);

            nodeEnter.append('circle')
                .attr('class', 'center-circle')
                .attr('r', radius)
                .on('click', toggleChildren);

            nodeEnter.append('svg')
                .attr('class', 'log-icon action-icon')
                .attr('viewBox', '0 0 20 20')
                .attr('width', 14)
                .attr('height', 14)
                .attr('y', -7)
                .html('<path d="M17.414 2.586a2 2 0 00-2.828 0L7 10.172V13h2.828l7.586-7.586a2 2 0 000-2.828z"></path><path fill-rule="evenodd" d="M2 6a2 2 0 012-2h4a1 1 0 010 2H4v10h10v-4a1 1 0 112 0v4a2 2 0 01-2 2H4a2 2 0 01-2-2V6z" clip-rule="evenodd"></path>')
                .on('click', (e, d) => {
                    if (d.data.name === "root") return;
                    e.stopPropagation();
                    showActionModal(d.data);
                });

            nodeEnter.append('text')
                .attr("dy", ".35em")
                .text(d => d.data.name)
                .on('click', (e, d) => {
                    if (d.data.name === "root") return;
                    e.stopPropagation();
                    showActionModal(d.data);
                });

            const nodeUpdate = nodeEnter.merge(node);
            nodeUpdate.transition().duration(duration)
                .attr("transform", d => "translate(" + d.y + "," + d.x + ")");

            nodeUpdate.style('visibility', d => d.depth === 0 ? 'hidden' : 'visible');

            nodeUpdate.select('.log-icon').attr('x', radius + 6);
            nodeUpdate.select('text').attr("x", radius + 24).attr("text-anchor", "start");

            const donutArc = d3.arc().innerRadius(radius).outerRadius(radius + borderWidth);
            const donutSlices = nodeUpdate.select('.donut-container').selectAll('path')
                .data(d => getNodeBorderData(d));
            donutSlices.enter().append('path')
                .merge(donutSlices)
                .attr('fill', d => d.color)
                .attr('d', donutArc);
            donutSlices.exit().remove();
            
            nodeUpdate.select('.center-circle').style('fill', '#ffffff');

            const nodeExit = node.exit().transition().duration(duration)
                .attr("transform", d => "translate(" + source.y + "," + source.x + ")").remove();
            nodeExit.selectAll('circle').attr('r', 1e-6);
            nodeExit.select('text').style('fill-opacity', 1e-6);

            const link = svg.selectAll('path.link').data(links, d => d.id);
            const linkEnter = link.enter().insert('path', "g")
                .attr("class", "link")
                .attr('d', d => {
                    const o = {x: source.x0, y: source.y0};
                    return d3.linkHorizontal()({source: o, target: o});
                });
            const linkUpdate = linkEnter.merge(link);
            linkUpdate.transition().duration(duration)
                .attr('d', d => d3.linkHorizontal()({source: {x: d.parent.x, y: d.parent.y}, target: {x: d.x, y: d.y}}));
            
            linkUpdate.style('visibility', d => d.depth === 1 ? 'hidden' : 'visible');
            
            link.exit().transition().duration(duration)
                .attr('d', d => {
                    const o = {x: source.x, y: source.y};
                    return d3.linkHorizontal()({source: o, target: o});
                })
                .remove();

            nodes.forEach(d => { d.x0 = d.x; d.y0 = d.y; });
        }

        function toggleChildren(event, d) {
            if (d.data.name === "root") return;
            if (d.children) {
                d._children = d.children;
                d.children = null;
            } else {
                d.children = d._children;
                d._children = null;
            }
            updateD3(d);
        }

        function expandAll() {
            if (!d3Root) return;
            d3Root.each(d => {
                if (d._children) {
                    d.children = d._children;
                    d._children = null;
                }
            });
            updateD3(d3Root);
        }

        function collapseAll() {
            if (!d3Root || !d3Root.children) return;
            d3Root.children.forEach(collapse);
            updateD3(d3Root);
        }

        // --- MODAL & EVENT HANDLER LOGIC ---
        function showActionModal(nodeData) {
            activeNodeForActions = nodeData;
            actionModal.title.textContent = `Actions for "${nodeData.name}"`;
            actionModal.el.classList.remove('hidden');
        }
        function hideActionModal() {
            actionModal.el.classList.add('hidden');
            activeNodeForActions = null;
        }
        actionModal.logBtn.onclick = () => { if (activeNodeForActions) showLogModal(activeNodeForActions); hideActionModal(); };
        actionModal.tasksBtn.onclick = () => { if (activeNodeForActions) showTaskModal(activeNodeForActions); hideActionModal(); };
        actionModal.editBtn.onclick = () => { if (activeNodeForActions) showEditModal(activeNodeForActions); hideActionModal(); };
        actionModal.aiCreatorBtn.onclick = () => { if (activeNodeForActions) showAICreatorModal(activeNodeForActions); hideActionModal(); };
        actionModal.goalBtn.onclick = () => { if (activeNodeForActions) showGoalModal(activeNodeForActions); hideActionModal(); };
        actionModal.historyBtn.onclick = () => { if (activeNodeForActions) showHistoryModal(activeNodeForActions); hideActionModal(); };
        actionModal.notesBtn.onclick = () => { if (activeNodeForActions) showNotesModal(activeNodeForActions); hideActionModal(); };
        actionModal.addBtn.onclick = () => { if (activeNodeForActions) showAddModal(activeNodeForActions); hideActionModal(); };
        actionModal.deleteBtn.onclick = () => { if (activeNodeForActions) showDeleteModal({ item: activeNodeForActions, type: 'node' }); hideActionModal(); };
        actionModal.closeBtn.onclick = hideActionModal;

        function showLogModal(node) { nodeToLog = node; logModal.subtitle.textContent = `For: ${node.name}`; logModal.input.value = ''; logModal.el.classList.remove('hidden'); }
        function hideLogModal() { logModal.el.classList.add('hidden'); }
        logModal.confirmBtn.onclick = () => { if (nodeToLog) { nodeToLog.logs.push({ timestamp: new Date().toISOString(), note: logModal.input.value.trim() }); logInteractionForToday(); saveData(); renderD3Tree(); } hideLogModal(); };
        logModal.cancelBtn.onclick = hideLogModal;

        function showNotesModal(node) { nodeForNotes = node; notesModal.title.textContent = `Category Notes for "${node.name}"`; notesModal.input.value = ''; renderNotesList(); notesModal.el.classList.remove('hidden'); }
        function hideNotesModal() { notesModal.el.classList.add('hidden'); }
        function renderNotesList() {
            notesModal.list.innerHTML = '';
            if (!nodeForNotes.notes || nodeForNotes.notes.length === 0) { notesModal.list.innerHTML = `<p class="text-gray-500 italic">No notes yet.</p>`; return; }
            [...nodeForNotes.notes].reverse().forEach(note => {
                const noteEl = document.createElement('div');
                noteEl.className = 'p-2 bg-gray-100 rounded-md flex justify-between items-start';
                noteEl.innerHTML = `<div><p class="text-sm text-gray-800">${note.text}</p><p class="text-xs text-gray-500 text-left mt-1">${new Date(note.timestamp).toLocaleString()}</p></div>`;
                const deleteIcon = document.createElement('div');
                deleteIcon.innerHTML = `<svg class="edit-icon remove-icon w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>`;
                deleteIcon.onclick = (e) => { e.stopPropagation(); showDeleteModal({ item: note, type: 'note', parent: nodeForNotes }); };
                noteEl.appendChild(deleteIcon);
                notesModal.list.appendChild(noteEl);
            });
        }
        notesModal.saveBtn.onclick = () => { const text = notesModal.input.value.trim(); if (text && nodeForNotes) { nodeForNotes.notes.push({ timestamp: new Date().toISOString(), text: text }); saveData(); renderNotesList(); renderD3Tree(); notesModal.input.value = ''; } };
        notesModal.closeBtn.onclick = hideNotesModal;
        
        function showHistoryModal(node) { nodeForHistory = node; historyModal.title.textContent = `Log History for "${node.name}"`; renderHistoryList(); historyModal.el.classList.remove('hidden'); }
        function hideHistoryModal() { historyModal.el.classList.add('hidden'); }
        function renderHistoryList() {
            historyModal.list.innerHTML = '';
            if (!nodeForHistory.logs || nodeForHistory.logs.length === 0) { historyModal.list.innerHTML = `<p class="text-gray-500 italic">No logs yet.</p>`; return; }
            [...nodeForHistory.logs].reverse().forEach(log => {
                const logEl = document.createElement('div');
                logEl.className = 'p-2 bg-gray-100 rounded-md';
                const noteHTML = log.note ? `<p class="text-sm text-gray-800 mt-1 pl-2 border-l-2 border-gray-300">${log.note}</p>` : '';
                logEl.innerHTML = `<p class="text-xs text-gray-500">${new Date(log.timestamp).toLocaleString()}</p>${noteHTML}`;
                historyModal.list.appendChild(logEl);
            });
        }
        historyModal.closeBtn.onclick = hideHistoryModal;

        function showGoalModal(node) { nodeForGoal = node; goalModal.title.textContent = `Set Goal for "${node.name}"`; goalModal.textInput.value = ''; goalModal.dateInput.value = ''; renderGoalList(); goalModal.el.classList.remove('hidden'); }
        function hideGoalModal() { goalModal.el.classList.add('hidden'); }
        function renderGoalList() {
            goalModal.list.innerHTML = '';
            if (!nodeForGoal.goals || nodeForGoal.goals.length === 0) { goalModal.list.innerHTML = `<p class="text-gray-500 italic">No goals yet.</p>`; return; }
            [...nodeForGoal.goals].reverse().forEach(goal => {
                const goalEl = document.createElement('div');
                goalEl.className = 'p-2 bg-gray-100 rounded-md flex justify-between items-start';
                goalEl.innerHTML = `<div><p class="text-sm text-gray-800">${goal.text}</p><p class="text-xs text-gray-500 text-left mt-1">Target: ${new Date(goal.targetDate).toLocaleDateString()}</p></div>`;
                const deleteIcon = document.createElement('div');
                deleteIcon.innerHTML = `<svg class="edit-icon remove-icon w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>`;
                deleteIcon.onclick = (e) => { e.stopPropagation(); showDeleteModal({ item: goal, type: 'goal', parent: nodeForGoal }); };
                goalEl.appendChild(deleteIcon);
                goalModal.list.appendChild(goalEl);
            });
        }
        goalModal.saveBtn.onclick = () => { const text = goalModal.textInput.value.trim(); const targetDate = goalModal.dateInput.value; if (text && targetDate && nodeForGoal) { nodeForGoal.goals.push({ text, targetDate, completed: false, createdAt: new Date().toISOString() }); saveData(); renderGoalList(); renderD3Tree(); goalModal.textInput.value = ''; goalModal.dateInput.value = ''; } };
        goalModal.closeBtn.onclick = hideGoalModal;

        function showAddModal(node) { nodeToAddTo = node; addModal.title.textContent = node ? `Add to "${node.name}"` : 'Add New Pillar'; addModal.input.value = ''; addModal.el.classList.remove('hidden'); }
        function hideAddModal() { addModal.el.classList.add('hidden'); }
        addModal.confirmBtn.onclick = () => { const name = addModal.input.value.trim(); if (name) { const newNode = { name: name, children: [], notes: [], logs: [], goals: [], tasks: [], createdAt: new Date().toISOString() }; if (nodeToAddTo) { if (!nodeToAddTo.children) nodeToAddTo.children = []; nodeToAddTo.children.push(newNode); } else { lifePillars.push(newNode); } saveData(); renderD3Tree(); renderTaskView(); } hideAddModal(); };
        addModal.cancelBtn.onclick = hideAddModal;

        function showDeleteModal(itemPayload) {
            itemToDelete = itemPayload;
            deleteModal.title.textContent = `Confirm Deletion`;
            let message = '';
            switch(itemToDelete.type) {
                case 'node': message = `Are you sure you want to delete "${itemToDelete.item.name}" and all of its contents?`; break;
                case 'note': message = `Are you sure you want to delete this note?`; break;
                case 'goal': message = `Are you sure you want to delete this goal?`; break;
            }
            deleteModal.text.textContent = message;
            deleteModal.el.classList.remove('hidden');
        }
        function hideDeleteModal() { deleteModal.el.classList.add('hidden'); }
        
        function findAndRemoveNode(targetNode, nodes) {
            for (let i = 0; i < nodes.length; i++) {
                if (nodes[i] === targetNode) {
                    nodes.splice(i, 1);
                    return true;
                }
                if (nodes[i].children) { if (findAndRemoveNode(targetNode, nodes[i].children)) return true; }
            }
            return false;
        }

        deleteModal.confirmBtn.onclick = () => {
            if (!itemToDelete) return;
            switch(itemToDelete.type) {
                case 'node': findAndRemoveNode(itemToDelete.item, lifePillars); break;
                case 'note': const noteIndex = itemToDelete.parent.notes.indexOf(itemToDelete.item); if (noteIndex > -1) itemToDelete.parent.notes.splice(noteIndex, 1); break;
                case 'goal': const goalIndex = itemToDelete.parent.goals.indexOf(itemToDelete.item); if (goalIndex > -1) itemToDelete.parent.goals.splice(goalIndex, 1); break;
            }
            saveData();
            renderD3Tree();
            renderTaskView();
            if (!notesModal.el.classList.contains('hidden')) renderNotesList();
            if (!historyModal.el.classList.contains('hidden')) renderHistoryList();
            if (!goalModal.el.classList.contains('hidden')) renderGoalList();
            hideDeleteModal();
        };
        deleteModal.cancelBtn.onclick = hideDeleteModal;

        addPillarBtn.onclick = () => showAddModal(null);
        expandAllBtn.onclick = expandAll;
        collapseAllBtn.onclick = collapseAll;

        // --- AI DIARY LOGIC ---
        aiDiaryBtn.onclick = () => {
            aiDiaryModal.el.classList.remove('hidden');
            aiDiaryModal.input.value = '';
            aiDiaryModal.suggestionsContainer.classList.add('hidden');
            aiDiaryModal.spinner.classList.add('hidden');
            aiDiaryModal.errorMessage.classList.add('hidden');
            aiDiaryModal.analyzeBtn.classList.remove('hidden');
            aiDiaryModal.confirmBtn.classList.add('hidden');
        };
        aiDiaryModal.cancelBtn.onclick = () => aiDiaryModal.el.classList.add('hidden');

        function getAllNodePaths(nodes, path = [], paths = []) {
            nodes.forEach(node => {
                const currentPath = [...path, node.name];
                paths.push(currentPath.join('/'));
                if (node.children && node.children.length > 0) {
                    getAllNodePaths(node.children, currentPath, paths);
                }
            });
            return paths;
        }

        aiDiaryModal.analyzeBtn.onclick = async () => {
            const entryText = aiDiaryModal.input.value.trim();
            if (!entryText) {
                aiDiaryModal.errorMessage.textContent = "Please write a diary entry first.";
                aiDiaryModal.errorMessage.classList.remove('hidden');
                return;
            }

            aiDiaryModal.spinner.classList.remove('hidden');
            aiDiaryModal.analyzeBtn.classList.add('hidden');
            aiDiaryModal.errorMessage.classList.add('hidden');
            aiDiaryModal.suggestionsContainer.classList.add('hidden');

            const nodePaths = getAllNodePaths(lifePillars);
            const prompt = `Given the diary entry below, identify which of the following life categories are relevant. Respond with a JSON array of strings, where each string is one of the provided category paths.

Diary Entry:
"${entryText}"

Available Categories:
${nodePaths.join('\n')}

Respond only with the JSON array.`;

            try {
                let chatHistory = [{ role: "user", parts: [{ text: prompt }] }];
                const payload = { contents: chatHistory };
                const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;
                
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });

                if (!response.ok) {
                    throw new Error(`API request failed with status ${response.status}`);
                }

                const result = await response.json();
                
                let suggestions = [];
                if (result.candidates && result.candidates.length > 0 && result.candidates[0].content && result.candidates[0].content.parts && result.candidates[0].content.parts.length > 0) {
                    const rawText = result.candidates[0].content.parts[0].text;
                    const jsonStringMatch = rawText.match(/\[.*\]/s);
                    if (jsonStringMatch) {
                        suggestions = JSON.parse(jsonStringMatch[0]);
                    } else {
                        throw new Error("Could not parse AI response.");
                    }
                } else {
                     throw new Error("No suggestions returned from AI.");
                }

                aiDiaryModal.suggestionsList.innerHTML = '';
                if (suggestions.length > 0) {
                    suggestions.forEach(path => {
                        const label = document.createElement('label');
                        label.className = 'flex items-center gap-2 p-2 bg-gray-50 rounded-md hover:bg-gray-100';
                        label.innerHTML = `<input type="checkbox" checked class="h-4 w-4 rounded border-gray-300 text-indigo-600 focus:ring-indigo-500" data-path="${path}"> <span class="text-sm text-gray-800">${path}</span>`;
                        aiDiaryModal.suggestionsList.appendChild(label);
                    });
                    aiDiaryModal.confirmBtn.classList.remove('hidden');
                } else {
                    aiDiaryModal.suggestionsList.innerHTML = `<p class="text-gray-500 italic">The AI didn't find any matching categories. You can still log this manually.</p>`;
                }

                aiDiaryModal.suggestionsContainer.classList.remove('hidden');

            } catch (error) {
                console.error("AI Analysis Error:", error);
                aiDiaryModal.errorMessage.textContent = "Sorry, there was an error analyzing your entry. Please try again.";
                aiDiaryModal.errorMessage.classList.remove('hidden');
                aiDiaryModal.analyzeBtn.classList.remove('hidden');
            } finally {
                aiDiaryModal.spinner.classList.add('hidden');
            }
        };

        aiDiaryModal.confirmBtn.onclick = () => {
            const entryText = aiDiaryModal.input.value.trim();
            const selectedPaths = Array.from(aiDiaryModal.suggestionsList.querySelectorAll('input:checked')).map(input => input.dataset.path);

            selectedPaths.forEach(pathString => {
                const pathArray = pathString.split('/');
                const node = findNodeByPath(pathArray);
                if (node) {
                    if (!node.logs) node.logs = [];
                    node.logs.push({ timestamp: new Date().toISOString(), note: entryText });
                }
            });
            logInteractionForToday();
            saveData();
            renderD3Tree();
            aiDiaryModal.el.classList.add('hidden');
        };
        
        // --- AI TASK LOGIC ---
        aiTaskBtn.onclick = () => {
            aiTaskModal.el.classList.remove('hidden');
            aiTaskModal.input.value = '';
            aiTaskModal.suggestionsContainer.classList.add('hidden');
            aiTaskModal.spinner.classList.add('hidden');
            aiTaskModal.errorMessage.classList.add('hidden');
            aiTaskModal.analyzeBtn.classList.remove('hidden');
            aiTaskModal.confirmBtn.classList.add('hidden');
        };
        aiTaskModal.cancelBtn.onclick = () => aiTaskModal.el.classList.add('hidden');

        aiTaskModal.analyzeBtn.onclick = async () => {
            const entryText = aiTaskModal.input.value.trim();
            if (!entryText) {
                aiTaskModal.errorMessage.textContent = "Please describe the tasks first.";
                aiTaskModal.errorMessage.classList.remove('hidden');
                return;
            }

            aiTaskModal.spinner.classList.remove('hidden');
            aiTaskModal.analyzeBtn.classList.add('hidden');
            aiTaskModal.errorMessage.classList.add('hidden');
            aiTaskModal.suggestionsContainer.classList.add('hidden');

            const nodePaths = getAllNodePaths(lifePillars);
            const today = new Date().toISOString().split('T')[0];
            const prompt = `Given the text below and that today's date is ${today}, identify distinct tasks, suggest a deadline for each in YYYY-MM-DD format, and categorize each one using the most relevant path from the provided list.

Text:
"${entryText}"

Available Categories:
${nodePaths.join('\n')}

Respond with a JSON array of objects, where each object has three keys: "task" (a string for the task description), "category" (a string for the chosen category path), and "deadline" (a string in YYYY-MM-DD format). For example: [{"task": "Book flight to Melbourne for next Friday", "category": "Hobbies/Travel", "deadline": "2025-08-15"}, {"task": "Prepare presentation for Wednesday meeting", "category": "Career/Current Job", "deadline": "2025-08-13"}]. Respond only with the JSON array.`;

            try {
                let chatHistory = [{ role: "user", parts: [{ text: prompt }] }];
                const payload = { contents: chatHistory };
                const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;
                
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });

                if (!response.ok) {
                    throw new Error(`API request failed with status ${response.status}`);
                }

                const result = await response.json();
                
                let suggestions = [];
                if (result.candidates && result.candidates.length > 0 && result.candidates[0].content && result.candidates[0].content.parts && result.candidates[0].content.parts.length > 0) {
                    const rawText = result.candidates[0].content.parts[0].text;
                    const jsonStringMatch = rawText.match(/\[.*\]/s);
                    if (jsonStringMatch) {
                        suggestions = JSON.parse(jsonStringMatch[0]);
                    } else {
                        throw new Error("Could not parse AI response.");
                    }
                } else {
                     throw new Error("No suggestions returned from AI.");
                }

                aiTaskModal.suggestionsList.innerHTML = '';
                if (suggestions.length > 0) {
                    suggestions.forEach(suggestion => {
                        const el = document.createElement('div');
                        el.className = 'task-suggestion-item flex justify-between items-center p-2 bg-gray-50 rounded-md';
                        el.innerHTML = `
                            <div>
                                <p class="font-semibold text-gray-800">${suggestion.task}</p>
                                <p class="text-sm text-gray-600">Category: <span class="font-medium">${suggestion.category}</span></p>
                            </div>
                            <div class="flex items-center gap-2">
                                <label class="text-sm text-gray-600">Deadline:</label>
                                <input type="date" value="${suggestion.deadline || today}" class="p-1 border border-gray-300 rounded-md">
                            </div>
                        `;
                        el.dataset.task = suggestion.task;
                        el.dataset.category = suggestion.category;
                        aiTaskModal.suggestionsList.appendChild(el);
                    });
                    aiTaskModal.confirmBtn.classList.remove('hidden');
                } else {
                    aiTaskModal.suggestionsList.innerHTML = `<p class="text-gray-500 italic">The AI couldn't identify any tasks. You can add them manually.</p>`;
                }

                aiTaskModal.suggestionsContainer.classList.remove('hidden');

            } catch (error) {
                console.error("AI Task Analysis Error:", error);
                aiTaskModal.errorMessage.textContent = "Sorry, there was an error analyzing your tasks. Please try again.";
                aiTaskModal.errorMessage.classList.remove('hidden');
                aiTaskModal.analyzeBtn.classList.remove('hidden');
            } finally {
                aiTaskModal.spinner.classList.add('hidden');
            }
        };

        aiTaskModal.confirmBtn.onclick = () => {
            const suggestions = aiTaskModal.suggestionsList.querySelectorAll('.task-suggestion-item');
            
            suggestions.forEach(el => {
                const taskText = el.dataset.task;
                const categoryPath = el.dataset.category;
                const deadlineInput = el.querySelector('input[type="date"]');
                const deadline = deadlineInput.value;
                const pathArray = categoryPath.split('/');
                const node = findNodeByPath(pathArray);

                if (node) {
                    if (!node.tasks) node.tasks = [];
                    node.tasks.push({
                        text: taskText,
                        completed: false,
                        deadline: deadline
                    });
                }
            });
            logInteractionForToday();
            saveData();
            renderD3Tree();
            renderTaskView();
            aiTaskModal.el.classList.add('hidden');
        };

        // --- AI CREATOR LOGIC ---
        function showAICreatorModal(contextNode = null) {
            aiCreatorContextNode = contextNode;
            aiCreatorModal.el.classList.remove('hidden');
            aiCreatorModal.input.value = '';
            aiCreatorModal.suggestionsContainer.classList.add('hidden');
            aiCreatorModal.spinner.classList.add('hidden');
            aiCreatorModal.errorMessage.classList.add('hidden');
            aiCreatorModal.analyzeBtn.classList.remove('hidden');
            aiCreatorModal.confirmBtn.classList.add('hidden');
        }
        aiCreatorBtn.onclick = () => showAICreatorModal(null);
        aiCreatorModal.cancelBtn.onclick = () => aiCreatorModal.el.classList.add('hidden');

        aiCreatorModal.analyzeBtn.onclick = async () => {
            const entryText = aiCreatorModal.input.value.trim();
            if (!entryText) {
                aiCreatorModal.errorMessage.textContent = "Please describe the new project or goal.";
                aiCreatorModal.errorMessage.classList.remove('hidden');
                return;
            }

            aiCreatorModal.spinner.classList.remove('hidden');
            aiCreatorModal.analyzeBtn.classList.add('hidden');
            aiCreatorModal.errorMessage.classList.add('hidden');
            aiCreatorModal.suggestionsContainer.classList.add('hidden');

            const nodePaths = getAllNodePaths(lifePillars);
            const prompt = `You are organizing a comma-separated list of tasks or project ideas into a hierarchical life dashboard.
Input list:
${entryText}

Rules:
Identify shared themes and group them under the most fitting parent categories.
Within each group, create nested subcategories for more specific themes or instruments, skills, locations, etc.
Keep unrelated themes in separate top-level branches.
The hierarchy can go multiple levels deep.
Use the most specific and natural naming for each node.
Do not merge different concepts into a single generic category unless they are clearly the same.
Output only valid JSON in the following structure:
[ { "name": "Category or Task Name", "children": [ { "name": "Subcategory or Task Name" }, { "name": "Another Subcategory", "children": [ { "name": "Task" } ] } ] } ]

Example:
Input: Music, making music, writing music, practicing piano, piano song Cloud Atlas, old piano songs, practise guitar
Output:
[ { "name": "Hobbies", "children": [ { "name": "Music", "children": [ { "name": "Making music" }, { "name": "Writing music" }, { "name": "Piano", "children": [ { "name": "Practicing piano" }, { "name": "Piano song - Cloud Atlas" }, { "name": "Old piano songs" } ] }, { "name": "Guitar", "children": [ { "name": "Practise guitar" } ] } ] } ] } ]`;

            try {
                let chatHistory = [{ role: "user", parts: [{ text: prompt }] }];
                const payload = { contents: chatHistory };
                const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;
                
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });

                if (!response.ok) {
                    throw new Error(`API request failed with status ${response.status}`);
                }

                const result = await response.json();
                
                let suggestions = [];
                if (result.candidates && result.candidates.length > 0 && result.candidates[0].content && result.candidates[0].content.parts && result.candidates[0].content.parts.length > 0) {
                    const rawText = result.candidates[0].content.parts[0].text;
                    const jsonStringMatch = rawText.match(/\[.*\]/s);
                    if (jsonStringMatch) {
                        suggestions = JSON.parse(jsonStringMatch[0]);
                    } else {
                        throw new Error("Could not parse AI response.");
                    }
                } else {
                     throw new Error("No suggestion returned from AI.");
                }

                aiCreatorModal.suggestionsList.innerHTML = buildSuggestionHtml(suggestions);
                aiCreatorModal.suggestionsContainer.classList.remove('hidden');
                aiCreatorModal.confirmBtn.classList.remove('hidden');


            } catch (error) {
                console.error("AI Creator Analysis Error:", error);
                aiCreatorModal.errorMessage.textContent = "Sorry, there was an error analyzing your request. Please try again.";
                aiCreatorModal.errorMessage.classList.remove('hidden');
                aiCreatorModal.analyzeBtn.classList.remove('hidden');
            } finally {
                aiCreatorModal.spinner.classList.add('hidden');
            }
        };
        
        function buildSuggestionHtml(items) {
            if (!items || items.length === 0) return '';
            let html = '<ul class="pl-4">';
            items.forEach(item => {
                html += `<li class="mt-1">
                    <label class="flex items-center gap-2">
                        <input type="checkbox" checked class="h-4 w-4 rounded border-gray-300 text-orange-600 focus:ring-orange-500">
                        <span class="font-medium">${item.name}</span>
                    </label>
                    ${buildSuggestionHtml(item.children)}
                </li>`;
            });
            html += '</ul>';
            return html;
        }

        aiCreatorModal.confirmBtn.onclick = () => {
            const rootList = aiCreatorModal.suggestionsList.querySelector('ul');

            function processList(listElement, parentNodes, rootNodes) {
                const items = Array.from(listElement.children);

                items.forEach(item => {
                    const label = item.querySelector('label');
                    if (!label) return;

                    const checkbox = label.querySelector('input[type="checkbox"]');
                    const nameSpan = label.querySelector('span');
                    const newNodeName = nameSpan.textContent;
                    const nestedList = item.querySelector('ul');
                    
                    if (checkbox && checkbox.checked) {
                        // This item is checked, create it in its designated parent list.
                        let node = parentNodes.find(n => n.name === newNodeName);
                        if (!node) {
                            node = {
                                name: newNodeName,
                                children: [],
                                notes: [],
                                logs: [],
                                goals: [],
                                tasks: [],
                                createdAt: new Date().toISOString()
                            };
                            parentNodes.push(node);
                        }
                        
                        // Process its children, which will be added to this new node.
                        if (nestedList) {
                            if (!node.children) node.children = [];
                            processList(nestedList, node.children, rootNodes);
                        }
                    } else {
                        // This item is NOT checked.
                        // Its children, if they are checked, should be treated as top-level nodes.
                        if (nestedList) {
                            processList(nestedList, rootNodes, rootNodes);
                        }
                    }
                });
            }

            if (rootList) {
                 if (aiCreatorContextNode) {
                    if (!aiCreatorContextNode.children) aiCreatorContextNode.children = [];
                    processList(rootList, aiCreatorContextNode.children, aiCreatorContextNode.children);
                } else {
                    processList(rootList, lifePillars, lifePillars);
                }
            }
            
            saveData();
            renderD3Tree();
            renderTaskView();
            aiCreatorModal.el.classList.add('hidden');
        };



        // --- IMPORT/EXPORT ---
        function findNodeByPath(pathArray, nodes = lifePillars) {
            let currentNode = { children: nodes };
            for (const name of pathArray) {
                if (!currentNode || !currentNode.children) return null;
                currentNode = currentNode.children.find(n => n.name === name);
            }
            return currentNode;
        }

        function findOrCreateNodeByPath(pathArray) {
            let currentNodes = lifePillars;
            let foundNode = null;

            for (let i = 0; i < pathArray.length; i++) {
                const nodeName = pathArray[i];
                let nextNode = currentNodes.find(n => n.name === nodeName);

                if (!nextNode) {
                    nextNode = { name: nodeName, children: [], notes: [], logs: [], goals: [], tasks: [], createdAt: new Date().toISOString() };
                    currentNodes.push(nextNode);
                }
                
                if (i === pathArray.length - 1) {
                    foundNode = nextNode;
                } else {
                    if (!nextNode.children) {
                        nextNode.children = [];
                    }
                    currentNodes = nextNode.children;
                }
            }
            return foundNode;
        }
        
        function CsvQuote(field) {
            const stringField = String(field === null || field === undefined ? '' : field);
            const escapedField = stringField.replace(/"/g, '""');
            return `"${escapedField}"`;
        }
        
        function performExport(filename) {
            const header = ["Path", "Type", "Timestamp", "Data1", "Data2", "Data3"];
            const rows = [];

            function traverseForExport(nodes, path = []) {
                nodes.forEach(node => {
                    const currentPath = [...path, node.name];
                    const pathString = currentPath.join('/');
                    
                    rows.push([pathString, 'Node', node.createdAt || '', '', '', '']);

                    if (node.logs) {
                        node.logs.forEach(log => {
                            rows.push([pathString, 'Log', log.timestamp, log.note || '', '', '']);
                        });
                    }
                    if (node.notes) {
                        node.notes.forEach(note => {
                            rows.push([pathString, 'Note', note.timestamp, note.text || '', '', '']);
                        });
                    }
                    if (node.goals) {
                        node.goals.forEach(goal => {
                            rows.push([pathString, 'Goal', goal.createdAt, goal.text || '', goal.targetDate, '']);
                        });
                    }
                    if (node.tasks) {
                        node.tasks.forEach(task => {
                            rows.push([pathString, 'Task', '', task.text, task.completed, task.deadline || '']);
                        });
                    }
                    if (node.children) {
                        traverseForExport(node.children, currentPath);
                    }
                });
            }

            traverseForExport(lifePillars);
            
            const csvRows = [
                header.join(','), 
                ...rows.map(row => row.map(CsvQuote).join(','))
            ];
            const csvContent = csvRows.join('\n');
            
            const link = document.createElement("a");
            link.setAttribute("href", URL.createObjectURL(new Blob([csvContent], { type: 'text/csv;charset=utf-8;' })));
            link.setAttribute("download", filename);
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
        }

        exportBtn.onclick = () => {
            const defaultFilename = `life_dashboard_export_${new Date().toISOString().split('T')[0]}`;
            exportModal.input.value = defaultFilename;
            exportModal.el.classList.remove('hidden');
        };

        exportModal.cancelBtn.onclick = () => {
            exportModal.el.classList.add('hidden');
        };

        exportModal.confirmBtn.onclick = () => {
            let filename = exportModal.input.value.trim();
            if (!filename) {
                filename = `life_dashboard_export_${new Date().toISOString().split('T')[0]}`;
            }
            if (!filename.toLowerCase().endsWith('.csv')) {
                filename += '.csv';
            }
            performExport(filename);
            exportModal.el.classList.add('hidden');
        };

        importBtn.onclick = () => importInput.click();

        importInput.onchange = (event) => {
            const file = event.target.files[0];
            if (!file) return;
            fileToImport = file;
            importConfirmModal.el.classList.remove('hidden');
            importInput.value = ''; 
        };

        importConfirmModal.cancelBtn.onclick = () => {
            fileToImport = null;
            importConfirmModal.el.classList.add('hidden');
        };

        importConfirmModal.confirmBtn.onclick = () => {
            if (!fileToImport) return;

            const reader = new FileReader();
            reader.onload = (e) => {
                lifePillars = []; // Clear current data
                const lines = e.target.result.trim().split('\n').slice(1);

                lines.forEach(line => {
                    if (line.trim() === '') return;
                    
                    const parts = [];
                    let currentField = '';
                    let inQuotes = false;

                    for (let i = 0; i < line.length; i++) {
                        const char = line[i];

                        if (char === '"' && (i === 0 || line[i-1] === ',')) {
                           if (!inQuotes) {
                             inQuotes = true;
                             continue;
                           }
                        }

                        if (inQuotes) {
                            if (char === '"') {
                                if (i + 1 < line.length && line[i + 1] === '"') {
                                    currentField += '"';
                                    i++; 
                                } else {
                                    inQuotes = false;
                                }
                            } else {
                                currentField += char;
                            }
                        } else {
                           if (char === ',') {
                                parts.push(currentField);
                                currentField = '';
                            } else {
                                currentField += char;
                            }
                        }
                    }
                    parts.push(currentField); 
                    
                    const [path, type, timestamp, data1, data2, data3] = parts;

                    if (!path) return;

                    const pathArray = path.split('/');
                    const node = findOrCreateNodeByPath(pathArray);

                    if (node) {
                        switch(type) {
                            case 'Node':
                                node.createdAt = timestamp;
                                break;
                            case 'Log':
                                if (!node.logs) node.logs = [];
                                node.logs.push({ timestamp, note: data1 || '' });
                                break;
                            case 'Note':
                                if (!node.notes) node.notes = [];
                                node.notes.push({ timestamp, text: data1 || '' });
                                break;
                            case 'Goal':
                                if (!node.goals) node.goals = [];
                                node.goals.push({ createdAt: timestamp, text: data1 || '', targetDate: data2 || '', completed: false });
                                break;
                            case 'Task':
                                if (!node.tasks) node.tasks = [];
                                node.tasks.push({ text: data1, completed: data2 === 'true', deadline: data3 || null });
                                break;
                        }
                    }
                });

                saveData();
                renderD3Tree();
                renderTaskView();

                const feedbackEl = document.getElementById('save-feedback');
                if (feedbackEl) {
                    feedbackEl.textContent = 'Import Complete!';
                    feedbackEl.classList.remove('opacity-0');
                    setTimeout(() => {
                        feedbackEl.classList.add('opacity-0');
                        setTimeout(() => { feedbackEl.textContent = 'Saved!'; }, 500);
                    }, 2000);
                }
            };
            reader.readAsText(fileToImport);
            
            fileToImport = null;
            importConfirmModal.el.classList.add('hidden');
        };

        // --- TASK MODAL LOGIC ---
        function showTaskModal(node) {
            nodeForTasks = node;
            taskModal.title.textContent = `Manage Tasks for "${node.name}"`;
            taskModal.deadlineInput.value = new Date().toISOString().split('T')[0];
            renderTaskList();
            taskModal.el.classList.remove('hidden');
        }

        function hideTaskModal() {
            taskModal.el.classList.add('hidden');
            nodeForTasks = null;
        }

        function renderTaskList() {
            taskModal.taskList.innerHTML = '';
            if (!nodeForTasks.tasks || nodeForTasks.tasks.length === 0) {
                taskModal.taskList.innerHTML = `<p class="text-gray-500 italic">No tasks yet.</p>`;
                return;
            }

            nodeForTasks.tasks.forEach((task, index) => {
                const taskEl = document.createElement('div');
                taskEl.className = 'flex items-center justify-between p-2 bg-gray-50 rounded-md';
                
                const label = document.createElement('label');
                label.className = 'flex items-center gap-2';
                const checkbox = document.createElement('input');
                checkbox.type = 'checkbox';
                checkbox.checked = task.completed;
                checkbox.className = 'h-4 w-4 rounded border-gray-300 text-cyan-600 focus:ring-cyan-500';
                checkbox.onchange = () => {
                    task.completed = checkbox.checked;
                };
                
                const span = document.createElement('span');
                span.textContent = task.text;
                span.className = task.completed ? 'line-through text-gray-500' : '';
                
                const deadlineText = document.createElement('span');
                deadlineText.className = 'text-xs text-gray-500 ml-2';
                deadlineText.textContent = task.deadline ? `(Due: ${new Date(task.deadline).toLocaleDateString()})` : '';

                label.appendChild(checkbox);
                label.appendChild(span);
                label.appendChild(deadlineText);

                const deleteBtn = document.createElement('button');
                deleteBtn.innerHTML = `<svg class="w-4 h-4 text-gray-400 hover:text-red-500" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>`;
                deleteBtn.onclick = () => {
                    nodeForTasks.tasks.splice(index, 1);
                    renderTaskList();
                };

                taskEl.appendChild(label);
                taskEl.appendChild(deleteBtn);
                taskModal.taskList.appendChild(taskEl);
            });
        }
        
        taskModal.addBtn.onclick = () => {
            const text = taskModal.taskInput.value.trim();
            const deadline = taskModal.deadlineInput.value;
            if (text) {
                if (!nodeForTasks.tasks) nodeForTasks.tasks = [];
                nodeForTasks.tasks.push({ text: text, completed: false, deadline: deadline || null });
                taskModal.taskInput.value = '';
                renderTaskList();
            }
        };

        taskModal.saveBtn.onclick = () => {
            if (nodeForTasks) {
                saveData();
                renderD3Tree();
                renderTaskView();
            }
            hideTaskModal();
        };
        
        taskModal.closeBtn.onclick = hideTaskModal;

        // --- TASK VIEW LOGIC ---
        function getAllTasks() {
            const allTasks = [];
            function traverse(nodes, path = []) {
                nodes.forEach(node => {
                    const currentPath = [...path, node.name];
                    if (node.tasks && node.tasks.length > 0) {
                        node.tasks.forEach(task => {
                            if (!task.completed) {
                                allTasks.push({
                                    taskRef: task,
                                    node: node,
                                    path: currentPath.join(' / ')
                                });
                            }
                        });
                    }
                    if (node.children) {
                        traverse(node.children, currentPath);
                    }
                });
            }
            traverse(lifePillars);
            return allTasks;
        }

        function calculateTaskPriority(taskWrapper) {
            const now = new Date();
            if (!taskWrapper.taskRef.deadline) {
                return { score: 4, days: Infinity }; // Lowest priority
            }
            const deadline = new Date(taskWrapper.taskRef.deadline);
            const timeRemaining = deadline - now;
            const daysRemaining = timeRemaining / (1000 * 60 * 60 * 24);

            if (daysRemaining < 0) return { score: 0, days: daysRemaining }; // Overdue
            if (daysRemaining <= 2) return { score: 1, days: daysRemaining }; // Urgent
            if (daysRemaining <= 7) return { score: 2, days: daysRemaining }; // High
            return { score: 3, days: daysRemaining }; // Normal
        }

        function renderTaskView() {
            const container = document.getElementById('task-view-container');
            container.innerHTML = '';

            const tasks = getAllTasks();

            tasks.sort((a, b) => {
                const priorityA = calculateTaskPriority(a);
                const priorityB = calculateTaskPriority(b);
                if (priorityA.score !== priorityB.score) {
                    return priorityA.score - priorityB.score;
                }
                return priorityA.days - priorityB.days;
            });

            if (tasks.length === 0) {
                container.innerHTML = `<p class="text-gray-500 italic text-center">No incomplete tasks. Great job!</p>`;
                return;
            }

            tasks.forEach(taskWrapper => {
                const taskEl = document.createElement('div');
                const priority = calculateTaskPriority(taskWrapper);
                let borderColorClass = 'border-gray-300';
                if (taskWrapper.taskRef.deadline) {
                     switch(priority.score) {
                        case 0: borderColorClass = 'border-red-500'; break;
                        case 1: borderColorClass = 'border-orange-400'; break;
                        case 2: borderColorClass = 'border-yellow-400'; break;
                        default: borderColorClass = 'border-green-500'; break;
                    }
                }

                taskEl.className = `flex items-center justify-between p-3 bg-white rounded-lg shadow-sm border-l-4 ${borderColorClass}`;

                const content = `
                    <div class="flex items-center gap-3">
                        <div class="task-checkbox h-5 w-5 rounded border border-gray-300 flex-shrink-0 flex items-center justify-center cursor-pointer transition-colors">
                            <svg class="w-4 h-4 text-green-500 hidden" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M5 13l4 4L19 7"></path></svg>
                        </div>
                        <div>
                            <p class="font-medium text-gray-800">${taskWrapper.taskRef.text}</p>
                            <p class="text-sm text-gray-500">${taskWrapper.path}</p>
                            ${taskWrapper.taskRef.deadline ? `<p class="text-xs text-gray-400">Deadline: ${new Date(taskWrapper.taskRef.deadline).toLocaleDateString()}</p>` : ''}
                        </div>
                    </div>
                `;
                taskEl.innerHTML = content;
                
                const checkbox = taskEl.querySelector('.task-checkbox');
                checkbox.onclick = () => {
                    const tick = checkbox.querySelector('svg');
                    tick.classList.remove('hidden');
                    checkbox.classList.add('bg-green-100', 'border-green-400');

                    setTimeout(() => {
                        taskWrapper.taskRef.completed = true;
                        saveData();
                        renderD3Tree();
                        renderTaskView();
                    }, 300);
                };

                container.appendChild(taskEl);
            });
        }

        // --- CLEAR ALL LOGIC ---
        clearAllBtn.onclick = () => {
            clearAllModal.el.classList.remove('hidden');
        };

        clearAllModal.cancelBtn.onclick = () => {
            clearAllModal.el.classList.add('hidden');
        };

        clearAllModal.confirmBtn.onclick = () => {
            lifePillars = [];
            dailyLogs = {};
            saveData();
            renderD3Tree();
            renderTaskView();
            clearAllModal.el.classList.add('hidden');
        };

        // --- CALENDAR LOGIC ---
        calendarBtn.onclick = () => {
            renderCalendar(currentCalendarDate.getFullYear(), currentCalendarDate.getMonth());
            calendarModal.el.classList.remove('hidden');
        };

        calendarModal.closeBtn.onclick = () => {
            calendarModal.el.classList.add('hidden');
        };
        
        calendarModal.prevBtn.onclick = () => {
            currentCalendarDate.setMonth(currentCalendarDate.getMonth() - 1);
            renderCalendar(currentCalendarDate.getFullYear(), currentCalendarDate.getMonth());
        };

        calendarModal.nextBtn.onclick = () => {
            currentCalendarDate.setMonth(currentCalendarDate.getMonth() + 1);
            renderCalendar(currentCalendarDate.getFullYear(), currentCalendarDate.getMonth());
        };

        function calculateStreak() {
            let streak = 0;
            const today = new Date();
            let currentDate = new Date(today);

            while (true) {
                const dateString = `${currentDate.getFullYear()}-${String(currentDate.getMonth() + 1).padStart(2, '0')}-${String(currentDate.getDate()).padStart(2, '0')}`;
                if (dailyLogs[dateString]) {
                    streak++;
                    currentDate.setDate(currentDate.getDate() - 1);
                } else {
                    break;
                }
            }
            calendarModal.streakCounter.textContent = streak;
        }

        function renderCalendar(year, month) {
            const grid = calendarModal.grid;
            grid.innerHTML = '';
            const monthYearStr = new Date(year, month).toLocaleString('default', { month: 'long', year: 'numeric' });
            calendarModal.monthYear.textContent = monthYearStr;

            const firstDay = new Date(year, month, 1).getDay();
            const daysInMonth = new Date(year, month + 1, 0).getDate();
            const today = new Date();

            const dayHeaders = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'];
            dayHeaders.forEach(day => {
                const dayEl = document.createElement('div');
                dayEl.className = 'font-semibold text-xs text-gray-500';
                dayEl.textContent = day;
                grid.appendChild(dayEl);
            });

            for (let i = 0; i < firstDay; i++) {
                grid.appendChild(document.createElement('div'));
            }

            for (let day = 1; day <= daysInMonth; day++) {
                const dayEl = document.createElement('div');
                dayEl.textContent = day;
                dayEl.className = 'calendar-day p-1 rounded-full';
                
                const dateString = `${year}-${String(month + 1).padStart(2, '0')}-${String(day).padStart(2, '0')}`;
                if (dailyLogs[dateString]) {
                    dayEl.classList.add('logged-day');
                }
                if (day === today.getDate() && month === today.getMonth() && year === today.getFullYear()) {
                    dayEl.classList.add('today');
                }
                grid.appendChild(dayEl);
            }
            calculateStreak();
        }

        // --- EDIT MODAL LOGIC ---
        function showEditModal(node) {
            nodeToEdit = node;
            const pathInfo = getNodePath(node);
            if(pathInfo) {
                editModal.nameInput.value = pathInfo.name;
                editModal.pathInput.value = pathInfo.parentPath.join('/');
            }
            editModal.el.classList.remove('hidden');
        }

        function hideEditModal() {
            editModal.el.classList.add('hidden');
            nodeToEdit = null;
        }
        
        function getNodePath(targetNode, nodes = lifePillars, path = []) {
            for (const node of nodes) {
                if (node === targetNode) {
                    return { name: node.name, parentPath: path };
                }
                if (node.children) {
                    const found = getNodePath(targetNode, node.children, [...path, node.name]);
                    if (found) return found;
                }
            }
            return null;
        }

        function findParentAndRemove(targetNode, nodes = lifePillars) {
            for (let i = 0; i < nodes.length; i++) {
                if (nodes[i] === targetNode) {
                    nodes.splice(i, 1);
                    return true;
                }
                if (nodes[i].children) {
                    if (findParentAndRemove(targetNode, nodes[i].children)) return true;
                }
            }
            return false;
        }

        editModal.confirmBtn.onclick = () => {
            if (!nodeToEdit) return;

            const newName = editModal.nameInput.value.trim();
            const newPath = editModal.pathInput.value.trim();

            if (!newName) return;

            // Remove the node from its old location
            findParentAndRemove(nodeToEdit);

            // Update its name
            nodeToEdit.name = newName;

            const newPathArray = newPath ? newPath.split('/') : [];

            if (newPathArray.length > 0) {
                const parentNode = findOrCreateNodeByPath(newPathArray);
                if (!parentNode.children) parentNode.children = [];
                parentNode.children.push(nodeToEdit);
            } else {
                lifePillars.push(nodeToEdit);
            }

            saveData();
            renderD3Tree();
            renderTaskView();
            hideEditModal();
        };

        editModal.cancelBtn.onclick = hideEditModal;


        // --- INITIALIZATION ---
        function setDate() {
            const dateEl = document.getElementById('current-date');
            if (!dateEl) return;
            const today = new Date();
            const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };
            dateEl.textContent = today.toLocaleDateString(undefined, options);
        }

        setDate();
        loadData();
        renderD3Tree();
        renderTaskView();
        window.addEventListener('resize', () => {
            renderD3Tree();
        });
    </script>
</body>
</html>
