# pps
my project
# -*- coding: utf-8 -*-
db.define_table('stocktype',
                Field('type_name'),
                format = '%(type_name)s')

db.define_table('breed',
                Field('breed_name'),
                format = '%(breed_name)s')

db.define_table('sex',
                Field('sex'),
                format = '%(sex)s')

db.define_table('measure',
                Field('unit_of_measure'),
                format = '%(unit_of_measure)s')

db.define_table('stocksingle',
                Field('tag_no'),
                Field('dob'))

db.define_table('task',
                Field('task'),
                format= '%(task)s'
                )

db.define_table('employee',
                Field('first_name'),
                Field('last_name'),
                Field('ph_number'),
                format = '%(employee)s'
                )

db.define_table('paddocks',
                Field('paddock_name'),
                Field('size'),
                Field('pasture'),
                Field('soil_type'),
                Field('comments', 'text'),
                format = '%(paddock_name)s'
                )

db.define_table('products',
                Field('product'),
                Field('type', requires=IS_IN_SET(['Drench','Spray'])),
                Field('date_purchased', 'date',requires = IS_DATE(format=('%d-%m-%Y'))),
                Field('expiry_date', 'date',requires = IS_DATE(format=('%d-%m-%Y'))),
                Field('batch_no'),
                Field('qty','integer', default =0),
                Field('remaining_qty','integer', default =0), # = products.qty - stocktasks.count*stocktasks.doserate or paddock.size*paddocktasks.rate
                Field('dose_rate'),
                Field('u_o_m', requires =IS_IN_DB(db, 'measure.unit_of_measure','%(unit_of_measure)s')),
                Field('with_holding','integer', default =0),
                Field('ESI_withholding','integer', default =0),
                Field('cost', 'float',default =0.00),
                Field('comments', 'text'),
                format=lambda r:'%s %s'%(r.product,r.batch_no),
                )

db.define_table('stock',
                Field('mob_id'),
                Field('acquistition',requires=IS_IN_SET(['Bred','Purchsed'])),
                Field('stock_type'), #requires = IS_IN_DB(db, 'stocktype.type_name', '%(type_name)s')),
                Field('breed'), #requires =IS_IN_DB(db, 'breed.breed_name', '%(breed_name)s')),
                Field('sex'), #requires =IS_IN_DB(db, 'sex.sex', '%(sex)s')),
                Field('description'),
                Field('year_of_drop', default='07/2012', requires=IS_MATCH('\d{2}/\d{4}')),
                Field('count','integer', default =0),# this is the starting count
                Field('count_now', 'integer'), # this count should come from the stocktasks.count
                Field('comments', 'text'),
                format = '%(mob_id)s'
                )

db.define_table('stocktasks',
                Field('started_date','date',default=request.now,requires = IS_DATE(format=('%d-%m-%Y'))),
                Field('completed_date','date',default=request.now,requires = IS_DATE(format=('%d-%m-%Y'))),
                Field('mob_id', 'reference stock'),#requires =IS_IN_DB(db, 'stock.mob_id', '%(mob_id)s',)),
                Field('count_now'), # need to update stock.count
                Field('task', 'reference task'),#requires = IS_IN_DB(db, 'task.task', '%(task)s')),'list:reference db.task', requires =[IS_IN_DB(db,'task.task', '%(task_task)s[%(groupby_task)s]',multiple=True)]),
                Field('estimated_weight','integer', default =0),
                Field('product', db.products),#requires =[IS_IN_DB(db,'products.product', '%(chemical_product)s[%(group_batch_no)s]',multiple=True)]),#'%(product)s
                #Field('batch_no',requires =IS_IN_DB(db, 'products.batch_no')),
                Field('dose_rate'), # This should use the stocktasks.estimated_weight x products.rate
                Field('count_now'), # need to update stock.count
                #Dont think these are needed here Field('cost_head', 'float', default =0.00), #chemical.cost/chemcial.u_o_m*stocktasks.doserate
                #Dont think these are needed here Field('cost_for_mob', 'float', default =0.00), #stocktaks.cost_head*stocktasks.count
                 Field('withhold_until', 'date',default=request.now, requires = IS_DATE(format=('%d-%m-%Y'))),#compute=lambda r: r[completed_date])),#requires = IS_DATE(format=('%d-%m-%Y'))),
                       #compute=lambda r: (r['stocktasks.completed_date']stock+r['r.products.product.with_holding']).date), # = stocktasks.completed_date + chemical.withholding
                #Dont think these are needed here Field('esi_withholding', 'date'), # stocktasks.completed_date + chemical.esi_withholding
                Field('paddock', 'reference paddocks'),# requires =IS_IN_DB(db, 'paddocks.paddock_name', '%(paddock_name)s')),
                #Dont think these are needed here Field('stocking_rate', 'float', default =0.00), # paddock.size/stock.count
                Field('employee',requires =IS_IN_DB(db, 'employee.first_name','%(first_name)s')),#[%(groupby_first_name)s]',multiple=True)]),
                #Field('completed_date','date',default=request.now),
                Field('comments', 'text'),
auth.signature),

db.define_table('paddocktasks',
                Field('paddock_name', 'reference paddocks'),#requires =IS_IN_DB(db, 'paddocks.paddock_name', '%(paddock_name)s')),
                Field('task','reference task'),# requires =[IS_IN_DB(db,'task.task', '%(task_task)s[%(task)s]',multiple=True)]), #requires = IS_IN_DB(db, 'task.first_name', '%(task)s')),
                Field('product',db.products),# requires =[IS_IN_DB(db,'products.product', '%(chemical_product)s[%(group_batch_no)s]',multiple=True)]),
                Field('rate'),
                #Field('mob_id', 'reference stock'),# this should be updated form the stocktasks.paddock
                #Dont think these are needed here Field('cost_per_ha', 'float', default =0.00), #paddocktasks.rate*products.
                #Dont think these are needed here Field('u_o_m', requires = IS_IN_DB(db, 'measure.unit_of_measure','%(unit_of_measure)s')),
                #Dont think these are needed here Field('withholding_until', 'date'), # = paddocktasks.finished_date + chemical.withholding
                #Dont think these are needed here Field('esi_withholding', 'date'), # = paddocktasks.finished_date + chemical.esi_withholding
                Field('employee',requires =IS_IN_DB(db, 'employee.first_name','%(first_name)s')),#'list:reference db.employee',requires =[IS_IN_DB(db, 'employee.first_name', '%(employee_first_name)s[%9groupby_firstname)s]',multiple=True)]),
                Field('start_date','date', default=request.now,requires = IS_DATE(format=('%d-%m-%Y'))),
                Field('finished_date', 'date', default=request.now,requires = IS_DATE(format=('%d-%m-%Y'))),
                Field('comments', 'text'),
                )
db.paddocktasks._plural = 'Paddock Tasks'
db.define_table('vendor',
                Field('name'),
                Field('address'),
                Field('address_1'),
                Field('phone'),
                Field('PIC_number'),
                Field('comments', 'text'),
                format = '%(name)s'
                )

db.define_table('feedpurchase',
                Field('received_date', 'date', default=request.now,requires = IS_DATE(format=('%d-%m-%Y'))),
                Field('name', requires =IS_IN_DB(db, 'vendor.name','%(name)s')),
                Field('feed_type'),
                Field('qty'),
                Field('source'),
                Field('reference_no'),
                Field('analysis_or_certifcates'),
                Field('storage_location'),
                Field('employee',requires =IS_IN_DB(db, 'employee.first_name','%(first_name)s')),
                Field('comments', 'text'),
                )
